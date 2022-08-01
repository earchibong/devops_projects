## Devops Tooling Website Solution

## Step One: Prepare NFS Server
- deploy a new EC2 instance with RHEL Linux 8 Operating System.
- attach 3 additional volumes in the same availability zone each with 10GB storage

![rhel](https://user-images.githubusercontent.com/92983658/182121870-99f13101-81fa-472f-a09f-78de6e180e9c.png)

** Configure the server **

- ssh into server
- inspect block devices to the server: `lsblk`

![inspect_block_devices](https://user-images.githubusercontent.com/92983658/182123096-d7676e61-9a81-4177-95ad-4209c9a6741d.png)

- inspect dev directory for block devices: `ls /dev/`

![inspect](https://user-images.githubusercontent.com/92983658/182123818-f497ec2c-931f-4b85-a25a-b39f7aae63b5.png)

- check mounts and free space on server: `df -h`

![free_space](https://user-images.githubusercontent.com/92983658/182123860-454ced09-0584-4690-b344-fbf781e93142.png)

- create a single partition on each of the 3 disks using `gdisk` utility: `sudo gdisk /dev/<block_device_name>`
  - print partition table: p
  - add a new partition: n
  - write table to disk and exit: w
- inspect partition of 3 disks : `lsblk`

![inspect_2](https://user-images.githubusercontent.com/92983658/182125946-8e778d79-ada5-48d0-87ae-dce3c3b76bac.png)

- check for available partitions:
  - install `lvm2` package: `sudo yum install lvm2`
  - `sudo lvmdiskscan`

![scan](https://user-images.githubusercontent.com/92983658/182126777-51777eaa-1cda-44cf-bcae-f210ab23b2f2.png)

- mark each of 3 disks as physical volumes (PVs) to be used by `LVM`
```

sudo pvcreate /dev/xvdb1
sudo pvcreate /dev/xvdc1
sudo pvcreate /dev/xvdd1

```

- verify physical volumes are created: `sudo pvs`

![verify_physical_volumes](https://user-images.githubusercontent.com/92983658/182127090-f36f87ee-da2c-4f51-98b4-dd2a149590b3.png)

- add all 3 PVs to a volume group (VG). Name the VG `webdata-vg`: `sudo vgcreate webdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1`
- Verify that your VG has been created: `sudo vgs`

![verfiy_group](https://user-images.githubusercontent.com/92983658/182127447-0e4c4c1f-9b4f-4793-8aed-c0c0c606eee7.png)

- create **3 logical volumes** using `lvcreate` utility:
  - verify available space: `sudo vgdisplay`
  - adjust size of volumes to meet available soace : ensure `-l 8G` for all volumes together do not exceed avaiable space shown in display
```

sudo lvcreate -n lv-opt -L 8G webdata-vg
sudo lvcreate -n lv-apps -L 8G webdata-vg
sudo lvcreate -n lv-logs -L 8G webdata-vg

```

- Verify logical Volume has been created: `sudo lvs`

![l-volumes](https://user-images.githubusercontent.com/92983658/182134644-04fc0a68-989e-4003-b8a5-1f20d6a39380.png)

- verify server configuration
  - `sudo vgdisplay -v #view complete setup - VG, PV, and LV`
  - `sudo lsblk`

![verify_server_config](https://user-images.githubusercontent.com/92983658/182135118-b5c7dc1a-63d1-4791-839e-60eec8f16cd6.png)

- format the logical volumes with `xfs` filesystem:
```
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs

```
- create mount directory `/mnt` for logical volumes: `sudo mkdir -p /mnt`
  - Mount `lv-apps` on `/mnt/apps`:
    -- `sudo mkdir -p /mnt/apps`
    --  `sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
  - Mount `lv-logs` on `/mnt/logs`:
    -- `sudo mkdir -p /mnt/logs`
    -- backup all the files in the log directory `/mnt/logs` into `/home/recovery/logs`: `sudo rsync -av /mnt/logs/. /home/recovery/logs/`
    -- `sudo mount /dev/webdata-vg/lv-logs /mnt/logs`
    -- Restore log files back into `/mnt/logs`: `sudo rsync -av /home/recovery/logs/. /mnt/logs`
  - Mount `lv-opt` on `/mnt/opt`:
    -- `sudo mkdir -p /mnt/opt`
    -- `sudo mount /dev/webdata-vg/lv-opt /mnt/opt`
  
  **Update /ETC/FSTAB File**
  
  - `sudo blkid`
  - `sudo vi /etc/fstab`
  - update mounts using `UUID` from `sudo blkid`
  
  ![uuid](https://user-images.githubusercontent.com/92983658/182140977-b98b39c6-47d8-439d-9bcb-d1f497cc0c3b.png)

- Test configuration and reload daemon
```

sudo mount -a
sudo systemctl daemon-reload

```
- Verify setup : `df -h`

![setup_verify](https://user-images.githubusercontent.com/92983658/182141415-7e5c92a9-81d2-44f9-95cd-1fcc44077a0b.png)

**Install NFS SERVER**

```

sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service

```

