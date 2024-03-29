# LAMP stack with remote Database And NFS Server

Project: deploy a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

In this project the following components will be used for the solution:
- **Infrastructure:** AWS
- **Webserver Linux:** Red Hat Enterprise Linux 8
- **Database Server:** Ubuntu 20.04 + MySQL
- **Storage Server:** Red Hat Enterprise Linux 8 + NFS Server
- **Programming Language:** PHP
- **Code Repository:** GitHub

<br>

## Labs
- <a href="https://github.com/earchibong/devops_projects/blob/main/tooling_website.md#part-one-prepare-nfs-server">Prepare NFS Server</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/tooling_website.md#part-two-set-up-database-server">Set Up Database Server</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/tooling_website.md#part-three-configure-web-server">Configure Web Server</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/tooling_website.md#add-data-to-database">Add Data To Database</a>

<br>

## Part One: Prepare NFS Server
### Requirements:
- deploy a new EC2 instance with RHEL Linux 8 Operating System.
- attach 3 additional volumes in the same availability zone each with 10GB storage

![rhel](https://user-images.githubusercontent.com/92983658/182121870-99f13101-81fa-472f-a09f-78de6e180e9c.png)

<br>

### Step One: Configure The NFS Server

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
  - **Mount `lv-apps` on `/mnt/apps`:**
   - `sudo mkdir -p /mnt/apps`
   -  `sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
  - **Mount `lv-logs` on `/mnt/logs`:**
   - `sudo mkdir -p /mnt/logs`
   - `sudo mount /dev/webdata-vg/lv-logs /mnt/logs`
  - **Mount `lv-opt` on `/mnt/opt`:**
   - `sudo mkdir -p /mnt/opt`
   - `sudo mount /dev/webdata-vg/lv-opt /mnt/opt`
  
  <br>
  
  ### Step Two: Update /ETC/FSTAB File
  
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

<br>

### Step Three: Install NFS

```

sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service

```
![nfs_install](https://user-images.githubusercontent.com/92983658/182148092-0a5eaf9e-a00c-4713-a9cd-601310370a4d.png)


- set up permission that will allow our Web servers to read, write and execute files on NFS:
```

sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service

```

- Export the mounts for webservers’ `subnet cidr` to connect as clients
  - check `subnet cidr`:
   - open your EC2 details in AWS web console and locate ‘Networking’ tab
   - click a Subnet ID link to open the VPC management console
   
 ![subnet_CIDR](https://user-images.githubusercontent.com/92983658/182153567-101cd7f3-cdca-4b22-8915-d9c060e981d3.png)
 

- Configure access to NFS for clients within the same subnet
```

sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv

```
![nfs_access](https://user-images.githubusercontent.com/92983658/182597158-7eb7d949-9d6f-4015-9675-bab58a6a5139.png)


- open NFS ports on server
  - check nfs port: `rpcinfo -p | grep nfs`
  - open ports in server security group with new inbound rule: `NFS | 2049 | subnet CIDR`
  - also open the following ports:
   - `TCP 111 subnet CIDR`
   - `UDP 111 subnet CIDR`
   - `UDP 2049 subnet CIDR`
 
 <br>
 
 ## Part Two: Set Up Database Server
 - Launch Ubuntu EC2 instance with named `DB_server`
 
 <br>
 
 ### Step One: Install MySQL server on database server
 ```
 
sudo apt update
sudo apt install mysql-server

```
- verify system status: `sudo systemctl status mysql`
- if not running, restart server and enable so it runs even after a reboot:
```

sudo systemctl restart mysql
sudo systemctl enable mysql

```
![mysql](https://user-images.githubusercontent.com/92983658/182589074-c57d9e75-2d8e-47a6-abcb-589110c150d0.png)

<br>

### Step Two: Configure Database
```

sudo mysql
CREATE DATABASE tooling;
CREATE USER `webaccess`@`<Web-Server-SUBNET-CIDR>` IDENTIFIED BY 'pass';
GRANT ALL ON tooling.* TO 'webaccess'@'<Web-Server-SUBNET-CIDR>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit

```
![tooling_database](https://user-images.githubusercontent.com/92983658/182588092-1bea131f-fffb-421c-9f07-824d588c5bfb.png)

- edit bind address:
  - `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf` (sudo vi /etc/my.cnf for redhat)
  - Navigate to the line that begins with the `bind-address` and set it to : `0.0.0.0` or `*`
  - if `bind address is not in the `my.cnf` file add the following : `bind-address = 0.0.0.0`
  - save and exit
  
 ![bind_address](https://user-images.githubusercontent.com/92983658/183647003-e3fbded0-555d-4b23-8856-e2ee5608a36e.png)
  - restart mysql: `sudo systemctl restart mysql`
 
- on Database server, open `port 3306`:
  - for `Type` choose `Mysql/Aurora`
  - for `source` input `webserver SUBNET-CIDR`

![database_port_3306](https://user-images.githubusercontent.com/92983658/182589862-c8b0f4f3-6a0c-435b-bfdb-350a83268201.png)

*note: confirm IP allowed to connect to DB_server: SELECT host FROM mysql.user WHERE User = 'insert usernname here'; only users from IP addresses shown can connect to this database if the following error comes up : “ERROR 1130 (HY000): Host ‘10.120.152.137’ is not allowed to connect to this MySQL server”..then the above will usually tell you which hosts are allowed to connect*


<br>

## Part Three: Configure Web Server

- launch a red hat instance
- install `nfs client`: `sudo yum install nfs-utils nfs4-acl-tools -y`
- Mount `/var/www/` and target the NFS server’s export for apps:
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

```
- verify that NFS was mounted successfully: `df -h`

![verfiy_nfs_mount](https://user-images.githubusercontent.com/92983658/182597693-47a17202-2e1c-443a-ad15-a9ae65255863.png)

- Make sure that the changes will persist on Web Server after reboot:
  - `sudo vi /etc/fstab`
  - add the following: `<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`
  - run the command: `sudo mount <NFS-Server-Private-IP-Address>:/mnt/apps` or `sudo mount /var/www`

![poersistent_changes](https://user-images.githubusercontent.com/92983658/182598305-da81954b-79c1-4ea4-bb3d-86173553c233.png)

- Install `Remi’s repository`, `Apache` and `PHP`
```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1

sudo systemctl restart httpd

```
- open `TCP port 80` on the Web Server

- On all websesrvers, locate the `log` folder for Apache on the `Web Server` and mount it to `NFS` server’s export for logs:
```

sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd

```
- Verify that NFS was mounted successfully by running : `df -h`

![df_httpd](https://user-images.githubusercontent.com/92983658/183918054-bdcdfc18-45fb-47a1-84b2-af96a0c3a154.png)


- Make sure that the changes will persist on Web Server after reboot:
  - `sudo vi /etc/fstab`
  - add the following: `<NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd nfs defaults 0 0`

![var_log_httpd](https://user-images.githubusercontent.com/92983658/183917956-a81c45a9-4842-4726-89b2-a881449b07f1.png)

**repeat the above steps for 2 new web servers**


- Verify that Apache files and directories are available on the Web Server in `/var/www` and also on the NFS server in `/mnt/apps`:
  - on webservers: `ls -la /var/www`
  - on nfs server: `ls -la /mnt/apps`
  - If you see the same files – it means NFS is mounted correctly

![verify_apache_nfs](https://user-images.githubusercontent.com/92983658/182607487-4e5903d2-7685-443b-b6af-bf3a73f4ce02.png)


  - on `webserver_1` create a file named `test.md` in `/var/www/html`
  - on `webserver_2` check if `test.md` exists in `/var/www/html`


- Deploy the tooling website’s code to the `Webserver_1`: 
  - install git on EC2 instance: `sudo yum install git -y`
  - `cd /var/www/html`
  - clone the repository from git to EC2 using `HTTPS`: `sudo git clone repo url`
  - follow steps to git clone <a href="https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository">here</a>

  ![clone](https://user-images.githubusercontent.com/92983658/182809894-7ef5d6a0-8e09-4cfc-beb0-a3cf2a102403.png)

  - move all the contents of `tooling` into `var/www/html`: 
  ```
  
  cd var/www/html
  sudo mv tooling/* .
  sudo rm -r tooling
  
  ```
  
  - copy contents of `html` folder in the `var/www/html` directory directly into `/var/www/html` :
  ```
  
  sudo mv html/* .
  sudo rm -r html
  
  ```
  - check that contents on `var/www/html` on `webserver_1` are the same on `webserver_2` and 'webserver_3`
  
  ![tooling](https://user-images.githubusercontent.com/92983658/184131151-124147b5-ada6-4793-8480-1de518f36fbb.png)

  - change permissions on `var/www/html/ directory: `sudo chown -R apache:apache /var/www/html && sudo chmod -R 777 /var/www/html`
  
  <br>
  
  ### Install Mysql Client on all webservers
  `sudo yum install mysql`
 
- update website configuration on `/var/www/html/functions.php` to connect to database:
  - `sudo vi /var/www/html/functions.php`
  - under `connect to database`: update database details
   - in the 1st placeholder: `database private ip`
   - 2nd placeholder: `database username`
   - 3rd placeholder: `database password`
 
 ![functions_php](https://user-images.githubusercontent.com/92983658/184134693-5a7e99a9-09c4-4ca2-ad2e-c2d94230cdee.png)


  - On `webserver_1` Apply `tooling-db.sql` script to your database using this command:
  ```
  
  cd /var/www/html
  mysql -h <database-private-ip> -u <db-username> -p <db-pasword> <da-name> < <path to script>
  example: mysql -h 172.31.8.127 -u webaccess -p pass tooling < /var/www/html/tooling-db.sql
  
  ```
  
  <br>
  
  ## Add Data To Database
  - log-in to database server: 
  - create admin user `myuser` with password `password:` :    
  - grant privildeges to `myuser`: 
  - create table and description:
  
  ```
  CREATE USER `myuser`@`%` IDENTIFIED BY 'password:';
  GRANT ALL ON tooling.* TO 'myuser'@'%';
  show databases;
  use <databaseName>;
  show tables;
  
  ```
  
  if tables empty, create new table: `users` with the following columns: (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status')
  ```
  
  create table users(
    -> id int,
    -> username varchar(255),
    -> password varchar(255),
    -> email varchar(255),
    -> user_type varchar(255),
    -> status varchar(255)
    -> );
  
  desc databaseName.tableName;
  
  ```
  
  ![tables](https://user-images.githubusercontent.com/92983658/184139181-57b28f0a-f67d-4798-a6c3-d083af1ad43d.png)
  
  
  - insert values into `users` table : `1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1'`
  
  
  ```
  
  insert into users
    -> values
    -> ( 1, 'myuser','5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1' );
  
  
  ```
  
  ![insert_into](https://user-images.githubusercontent.com/92983658/184142061-bd506bb4-657f-41f5-a059-96f7cfa415d6.png)


  - log into database from webserver: `mysql -h <databse-private-ip> -u <db-username> -p`
  - see all data in users table:
  
  ```
  use databaseName;          example: use tooling;
  select * from tableName;   example: SELECT * from users;
  
  ```
  
  ![select_data](https://user-images.githubusercontent.com/92983658/184142648-189b9afc-36f5-4997-b5e5-032225ecf638.png)

  
  - test all three browsers:
  - Open the website in your browser `http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php` 
  - login into the website with myuser user.
  - if connection is refused while trying to test web server ip:
    - edit SELinux configuration: `sudo vi /etc/sysconfig/selinux`:
     - find the line `SELINUX=enforcing` and change to `SELINUX=disabled`
     - type the following: `sudo setenforce 0`
     - restart `httpd` 
 
 
![web](https://user-images.githubusercontent.com/92983658/184147660-634192e5-dc11-4abb-8ac2-96ab806c5d92.png)


![website](https://user-images.githubusercontent.com/92983658/184146967-5ca601e9-6967-4da4-ad67-ea032ed6c1e8.png)


<br>
