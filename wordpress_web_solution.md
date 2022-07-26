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

![DB_logical_volume](https://user-images.githubusercontent.com/92983658/180644300-705ac101-d80e-45a6-ab23-6938c29d49d2.png)

## Step Five: Install Wordpress on Web Server

- update repository: `sudo yum -y update`
- install wget, Apache and dependencies: `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`
- start Apache :
```

sudo systemctl enable httpd
sudo systemctl start httpd

```
- test webserver: open browser and access public DNS name: `http://ec2-13-41-188-119.eu-west-2.compute.amazonaws.com/`

![public_dns](https://user-images.githubusercontent.com/92983658/180831598-c5e47a9d-cad4-4039-a077-e9a9e5990b7d.png)


- install PHP and dependencies:
```

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1

```
- restart Apache: `sudo systemctl restart httpd`
- test PHP installation:
  - create a page for the test:
  ```
  
  cd /var/www/html
  sudo touch test.php
  sudo vi test.php
  
  ```
  - Type i to start the insert mode
  - Type <?php phpinfo() ?>
  - Type :wq to write the file and quit vi
- Open a browser and access `test.php` to test PHP installation: `http://ec2-13-41-188-119.eu-west-2.compute.amazonaws.com/`

![PHP_test](https://user-images.githubusercontent.com/92983658/180834760-06891f05-f984-4a42-877b-af1bc8a0efa3.png)


  
- download wordpress and copy to `var/www/html`:
```

mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
  sudo cp -R wordpress /var/www/html/
  
 ```
 ![wordpress](https://user-images.githubusercontent.com/92983658/180645032-30799f5c-9431-497d-ab53-a0a6d38cbd10.png)

- Configure SELinux Policies:
```

sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1

```

![SELinux](https://user-images.githubusercontent.com/92983658/180645137-9d03bd3a-c7a3-4a9a-9614-56de420b6bf3.png)


## Step Six: Install Mysql on Database Server
```

sudo yum update
sudo yum install mysql-server

```
- verify system status: `sudo systemctl status mysqld`
- if not running, restart server and enable so it runs even after a reboot:
```

sudo systemctl restart mysqld
sudo systemctl enable mysqld

```
![mysql_restart_enable_status](https://user-images.githubusercontent.com/92983658/180645620-83cccafd-7536-4fae-9654-df3b03c28a08.png)



## Step Seven: Configure Database Server To Work With Wordpress

```

sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit

```
![database_configure](https://user-images.githubusercontent.com/92983658/180645871-caa575e2-7f86-460f-a45d-2a02f1481244.png)


- on Database server, open `port 3306`:
  - for `Type` choose `Mysql/Aurora`
  - for `source` input `webserver private ip address/32`

![security_group](https://user-images.githubusercontent.com/92983658/180646072-8f4dc5bc-eb05-4093-83ad-8712736fb9ce.png)


*note: confirm IP allowed to connect to DB_server: `SELECT host FROM mysql.user WHERE User = 'insert usernname here';`*
*only users from IP addresses shown can connect to this database*
*if the following error comes up : “ERROR 1130 (HY000): Host ‘10.120.152.137’ is not allowed to connect to this MySQL server”..then the above will usually tell you which hosts are allowed to connect*


## Step Eight: Configure Wordpress To Connect To Remote Database

- on webserver, install `mysql client`: `sudo yum install mysql`
- test connection to database: `sudo mysql -u myuser -p -h <DB-Server-Private-IP-address>`

![test_connection](https://user-images.githubusercontent.com/92983658/180655091-f894e9ec-d6f7-49b0-baf2-ef128f5d80f4.png)

- Update the Database credentials detail on the wordpress configuration file in `var/www/html`
  - `cd var/www/html/wordpress`
  - `sudo vi wp-config.php`
  -  follow prompts to update database details in `wp-config` file using details from step 7:
    - `DB_USER`: user created in step 7 `myuser`
    - `DB_NAME`: `wordpress` 
    - `DB_PASSWORD`: `mypass`
    - `DB_HOSTNAME`: `DB_server private IP`
 
  ![wp_config](https://user-images.githubusercontent.com/92983658/181012820-7f5e34fc-597a-4f15-9e1e-8ecf750b74e6.png)


## Step Nine: Configure Apache For Wordpress

- in web server security group, edit inbound rules:
  - for `type`: enable port 80
  - for `source`: `custom 0.0.0.0/0`
- access wordpress from browser using webserver local ip: `http://<Web-Server-Public-DNS-Address>/wordpress/`

![wordpress](https://user-images.githubusercontent.com/92983658/181013127-64e50f08-100c-4b18-ace0-d2538963f038.png)

![wordpress_2](https://user-images.githubusercontent.com/92983658/181014533-d53ca1a0-5afc-4cfa-9eae-c7d43f12abe3.png)

![wordpress_3](https://user-images.githubusercontent.com/92983658/181014603-0e52fb83-af61-4ebf-b278-ac01e083b59e.png)


