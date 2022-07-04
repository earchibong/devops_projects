## STEP ONE: Launch a virutal machine, select machine specs, connect to instance with SSH

![Screenshot 2022-07-04 at 16 03 40](https://user-images.githubusercontent.com/92983658/177186020-93ecbe39-1229-4ade-96ac-e48e5f00fed4.png)
![Screenshot 2022-07-04 at 16 04 14](https://user-images.githubusercontent.com/92983658/177186027-4dc6d0ca-82cd-40ef-b9bf-41b15f160791.png)
![Screenshot 2022-07-04 at 16 04 24](https://user-images.githubusercontent.com/92983658/177186035-6e64e691-d4f3-4079-9598-0ed275ae7c91.png)

## STEP TWO: Connect to instance with SSH
`ssh -i "private.pem" ubuntu@ec2-13-40-174-165.eu-west-2.compute.amazonaws.com`

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
<br>
## STEP FOUR: Install PHP on EC2 Ubuntu
- install PHP with its common extensions:
` sudo apt-get install php libapache2-mod-php php-mysql php-curl php-gd php-json php-zip php-mbstring`

- confirm PHP version:
`php -v`
![Screenshot 2022-07-04 at 14 30 13](https://user-images.githubusercontent.com/92983658/177193126-140b605c-a344-4936-8598-64a002b898e1.png)
<br>
## STEP FIVE: Create Virtual Host

## STEP SIX: Restart Apache Web ServerServer
