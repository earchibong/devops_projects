## STEP ONE: Launch a virutal machine, select machine specs, connect to instance with SSH

![Screenshot 2022-07-04 at 16 03 40](https://user-images.githubusercontent.com/92983658/177186020-93ecbe39-1229-4ade-96ac-e48e5f00fed4.png)
![Screenshot 2022-07-04 at 16 04 14](https://user-images.githubusercontent.com/92983658/177186027-4dc6d0ca-82cd-40ef-b9bf-41b15f160791.png)
![Screenshot 2022-07-04 at 16 04 24](https://user-images.githubusercontent.com/92983658/177186035-6e64e691-d4f3-4079-9598-0ed275ae7c91.png)

## STEP TWO: Connect to instance with SSH
- `ssh -i "private.pem" ubuntu@ec2-13-40-174-165.eu-west-2.compute.amazonaws.com`


## STEP THREE: Install Apache
- update local package index of Ubuntu repositories : 
`sudo apt update`

- install apache server :
`sudo apt install apache2`

- verify apache installation :
`apache2 -version`

- list ufw application profiles :
`sudo ufw app list`
![Screenshot 2022-07-04 at 14 00 57](https://user-images.githubusercontent.com/92983658/177189906-56acccd0-c347-4cbe-86b3-bb5463d10a7e.png)

- select `Apache Full` from profiles list:
 `sudo ufw allow 'Apache Full'`
 
 - verify apache service is running:
 -- `curl http://localhost:80`
 
 ![Screenshot 2022-07-04 at 13 56 21](https://user-images.githubusercontent.com/92983658/177190000-9fe2eb44-382b-4e03-85ac-c5b15a4a2c56.png)

 <br>
 -- `sudo systemctl status apache2`
 ![Screenshot 2022-07-04 at 14 07 43](https://user-images.githubusercontent.com/92983658/177190199-19fbbe14-9f1a-4bdf-9f1f-584499bd10ba.png)
<br>
-- check public domain with AWS EC2 instance public IP addess

![Screenshot 2022-07-04 at 14 03 35](https://user-images.githubusercontent.com/92983658/177190361-c86eb711-69fc-4444-bef0-c989bd9aecb4.png)


## STEP FOUR: Install MYSQL on AWS EC2 Ubuntu from SSH

- `sudo apt-get install mysql-server` on SSH terminal
- `$ sudo mysql`
![Screenshot 2022-07-04 at 14 15 24](https://user-images.githubusercontent.com/92983658/177191148-d0bc9731-85d8-440b-bfb9-acf4154d2ec6.png)
<br>

- define password:
- `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

- `sudo mysql_secure_installation`

![Screenshot 2022-07-04 at 14 16 58](https://user-images.githubusercontent.com/92983658/177191815-e880828d-ef19-47c1-8fed-da8d6b2d6aee.png)

- test MYSQL console: 
`sudo mysql -u root -p`
![Screenshot 2022-07-04 at 14 20 44](https://user-images.githubusercontent.com/92983658/177192320-2070770a-a4ea-4b5b-8afc-4a0a9819aa2f.png)

- `exit`


## STEP FIVE: Install PHP on EC2 Ubuntu
- install PHP with its common extensions:
` sudo apt-get install php libapache2-mod-php php-mysql php-curl php-gd php-json php-zip php-mbstring`

- confirm PHP version:
`php -v`
![Screenshot 2022-07-04 at 14 30 13](https://user-images.githubusercontent.com/92983658/177193126-140b605c-a344-4936-8598-64a002b898e1.png)


## STEP SIX: Create Virtual Host for website using APACHE
- create a new directory `projectlamp` in `/var/www/`:
 `sudo mkdir /var/www/projectlamp`
- assign ownership of directory with crrent system user
`sudo chown -R $USER:$USER /var/www/projectlamp`
- create and open a new configuration file in Apache’s sites-available directory
` sudo nano /etc/apache2/sites-available/projectlamp.conf `

![Screenshot 2022-07-04 at 14 36 29](https://user-images.githubusercontent.com/92983658/177193901-913275bd-1b8d-46d8-8120-35f337112d94.png)

- paste configuration and exit with `ctrl - X`

![Screenshot 2022-07-04 at 17 45 31](https://user-images.githubusercontent.com/92983658/177194395-6f42cb15-47d7-4099-b496-ed14afd7e61e.png)

- enable a new virtual host:
`sudo a2ensite projectlamp`

- check for errors:
 `sudo apache2ctl configtest`
 
- reload Apache to confirm changes:
`sudo systemctl reload apache2`

- create an index.html file in `/var/www/projectlamp/index.html`:
`sudo nano /var/www/projectlamp/index.html `
- paste in test content
`Hello LAMP from hostname <public DNS address here> with public IP <public IP address here>`
- `ctrl - X` to exit
- type `y` to save

![Screenshot 2022-07-04 at 15 44 20](https://user-images.githubusercontent.com/92983658/177195615-03c4bfba-1fd7-46c6-ad59-be8b460f5aea.png)

- check public IP address
![Screenshot 2022-07-04 at 15 27 01](https://user-images.githubusercontent.com/92983658/177195815-b212a0e1-924d-4f1d-a06d-98d818c0df25.png)

## STEP SEVEN: Enable PHP on website
- configure `dir.conf` file:
`sudo nano /etc/apache2/mods-enabled/dir.conf`

- move PHP index file `index.php` to the first position after `Directory Index` 
![Screenshot 2022-07-04 at 15 45 26](https://user-images.githubusercontent.com/92983658/177196467-ef74912b-2f81-4a0c-b241-c12a6faba7b0.png)

*test PHP installation:*
- create `index.php` in root folder:
`nano /var/www/projectlamp/index.php`

- paste the following:
`<?php
phpinfo();`

- exit and save
- refresh the webpage

![Screenshot 2022-07-04 at 15 48 49](https://user-images.githubusercontent.com/92983658/177197231-5cd13ca8-9b17-482d-8cc9-dbc6683a52e0.png)

- delete the index.php file once finished checking  PHP details
`sudo rm /var/www/projectlamp/index.php`


- restart Apache webserver
`sudo service apache2 restart`

