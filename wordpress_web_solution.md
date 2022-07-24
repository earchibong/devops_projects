## DEPLOY A FULL SCALE THREE-TIER WEB SOLUTION WITH WORDPRESS CONTENT MANAGEMENT
## Step One: Launch Web Server

- Launch an EC2 instance with 3 additional volumes in the same availability zone each with 10GB storage

![WEB](https://user-images.githubusercontent.com/92983658/180609006-7764b76d-4760-4e84-9a76-b395431e49cf.png)


## Step Two: Configure Server

- open server to begin configuration
- inspect what block devices are attached to the server: `lsblk`

![INSPECT](https://user-images.githubusercontent.com/92983658/180609155-67e8fe46-3b1e-481b-a271-5480802a2bdb.png)

- inspect `dev directory` for block devices: `ls /dev/`

![DEV](https://user-images.githubusercontent.com/92983658/180609271-a8a5146f-b07d-4a4f-9e39-761faecae61e.png)

- check all mounts and free space on your server: `df -h`

![MOUNTS](https://user-images.githubusercontent.com/92983658/180609340-60b74ea4-241c-4e60-8935-068f6e7c87dd.png)

- create a single partition on each of the 3 disks using `gdisk` utility: `sudo gdisk /dev/<block_device_name>`

![GDISK_PARTITION](https://user-images.githubusercontent.com/92983658/180610434-df89f9a3-6fb6-4c3d-ad93-bdd9e00f76fd.png)


- inspect partition of 3 disks : `lsblk`

![PARTION_INSPECT](https://user-images.githubusercontent.com/92983658/180610485-94108938-70a6-428a-91aa-15ccd7166237.png)

- check for available partitions:
  - install `lvm2` package: `sudo yum install lvm2`
  - `sudo lvmdiskscan`
  
  ![AVAILABLE_PARTITIONS](https://user-images.githubusercontent.com/92983658/180610617-efb3238e-5036-4128-baea-c224ca16fb30.png)

- mark each of 3 disks as physical volumes (PVs) to be used by `LVM`
```

sudo pvcreate /dev/xvdb1
sudo pvcreate /dev/xvdc1
sudo pvcreate /dev/xvdd1

```

![PHYSICAL_VOLUMES](https://user-images.githubusercontent.com/92983658/180610720-7b9826e4-c9aa-485a-bc97-8343d3c1c91f.png)

- verify physical volumes are created: `sudo pvs`

![VERIFY](https://user-images.githubusercontent.com/92983658/180610775-e68188c4-f6bb-4697-8ebd-b0baa705d894.png)

- add all 3 PVs to a volume group (VG). Name the VG `webdata-vg`: `sudo vgcreate webdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1`
- Verify that your VG has been created: `sudo vgs`

![vgs_verify](https://user-images.githubusercontent.com/92983658/180611044-6e565d8a-bf48-491e-a2de-49c6b36d8b85.png)

- create 2 logical volumes using `lvcreate` utility: 
```

sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg

```

- Verify logical Volume has been created: `sudo lvs`

![lvs](https://user-images.githubusercontent.com/92983658/180611205-d0a6f616-83f0-4c68-9330-11eb139794e5.png)

- Verify Server Configuration
  - `sudo vgdisplay -v #view complete setup - VG, PV, and LV`
  - `sudo lsblk` 

![Screenshot 2022-07-23 at 16 21 00](https://user-images.githubusercontent.com/92983658/180611426-1833cd8b-0889-43ba-9c45-5dd30aee1418.png)

![lsblk](https://user-images.githubusercontent.com/92983658/180611431-338436fd-2beb-41cf-8068-f28dc5245f2b.png)

- format the logical volumes with `ext4` filesystem: 
```

sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

```

![ext4](https://user-images.githubusercontent.com/92983658/180611561-33963592-9463-4f29-87fe-3f353a25115e.png)

- Create `/var/www/html` directory to store website files: `sudo mkdir -p /var/www/html`
- Create `/home/recovery/logs` to store backup of log data: `sudo mkdir -p /home/recovery/logs`
- Mount `/var/www/html` on apps-lv logical volume: `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`
- backup all the files in the log directory `/var/log into /home/recovery/logs` with `rsync` utility: `sudo rsync -av /var/log/. /home/recovery/logs/`
-  Mount `/var/log` on `logs-lv` logical volume: `sudo mount /dev/webdata-vg/logs-lv /var/log`
-  Restore log files back into `/var/log` directory: `sudo rsync -av /home/recovery/logs/. /var/log`

![varwww](https://user-images.githubusercontent.com/92983658/180612044-93220e45-3ae2-4772-ab63-14af1c74b854.png)


## Step Three: Update `/ETC/FSTAB` File

-  `sudo blkid`
-  `sudo vi /etc/fstab`
  - update wordpress mounts using UUID from `sudo blkid`  

![wordpress_mounts](https://user-images.githubusercontent.com/92983658/180612756-dbc8a7d6-e28a-462e-a2db-14dedd43f1fe.png)

- Test configuration and reload daemon:
```
sudo mount -a
sudo systemctl daemon-reload
 
```

- Verify setup : `df -h`

![verify_setup](https://user-images.githubusercontent.com/92983658/180612888-e62e08c5-6ab3-479d-9646-0619b76d91b5.png)


## Step Four: Create Database Server
- Launch redhat EC2 instance with named `DB server`
- repeat the above. But,
  - instead of creating logical volume named `app-lv` create one named `db-lv`
  - mount `db-lv` to `/db` instead of `/var/www/html` 

