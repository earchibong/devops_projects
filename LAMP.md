# LAMP Stack Deployment

The LAMP stack is a popular software stack used for building and hosting dynamic websites and web applications. LAMP stands for Linux, Apache, MySQL, and PHP, which are the key components of this stack. Each component plays a specific role in enabling the stack to function together seamlessly.

Here's a brief explanation of each component:

**Linux:** Linux is the operating system that forms the foundation of the LAMP stack. It provides the core infrastructure and manages system resources. Linux distributions such as Ubuntu, CentOS, or Debian are commonly used in LAMP stack setups.

**Apache:** Apache is a web server software that handles incoming HTTP requests and serves web pages to clients. It is highly reliable, scalable, and widely used. Apache interprets requests, retrieves the requested files, and sends the appropriate response back to the client.

**MySQL:** MySQL is a popular open-source relational database management system (RDBMS). It provides a robust and efficient way to store, manage, and retrieve data. MySQL is often used as the LAMP stack's backend database for web applications.

**PHP:** PHP is a server-side scripting language that is used for creating dynamic web pages and web applications. It allows you to embed PHP code within HTML, enabling you to generate dynamic content and interact with the database. PHP is executed on the server, and the resulting HTML is sent to the client's browser.

When combined, these components work together to create a powerful and flexible environment for developing and deploying web applications. Here's how the components interact:

- The Apache web server listens for incoming HTTP requests on a specific port (usually port 80). It receives requests from clients, such as web browsers, and forwards them to the appropriate destination.
- When a request is received for a PHP file, Apache passes the request to the PHP interpreter.
- The PHP interpreter executes the PHP code embedded within the file. This code can interact with the MySQL database to retrieve or manipulate data.
- After executing the PHP code, the PHP interpreter generates HTML content as output.
- Apache sends the generated HTML back to the client, which renders it in the web browser.

The LAMP stack provides a robust and flexible platform for hosting dynamic websites and web applications. It is widely used for various purposes, ranging from small personal websites to large-scale enterprise applications. Additionally, it is highly customizable, allowing developers to extend and modify its components according to their specific requirements.

<br>

<br>

## Project Steps
- <a href=" ">Launch Virtual Machine</a>
- <a href=" ">Install Apache</a>
- <a href=" ">Install MYSQL from SSH</a>
- <a href=" ">Install PHP </a>
- <a href=" ">Create Virtual Host for a website using APACHE</a>
- <a href=" ">Enable PHP on website</a>

<br>

<br>

## Launch a virutal machine, select machine specs, connect to instance with SSH

![Screenshot 2022-07-04 at 16 03 40](https://user-images.githubusercontent.com/92983658/177186020-93ecbe39-1229-4ade-96ac-e48e5f00fed4.png)
![Screenshot 2022-07-04 at 16 04 14](https://user-images.githubusercontent.com/92983658/177186027-4dc6d0ca-82cd-40ef-b9bf-41b15f160791.png)
![Screenshot 2022-07-04 at 16 04 24](https://user-images.githubusercontent.com/92983658/177186035-6e64e691-d4f3-4079-9598-0ed275ae7c91.png)

<br>

<br>

### Connect to instance with SSH
```
ssh -i "private.pem" ubuntu@ec2-13-40-174-165.eu-west-2.compute.amazonaws.com

```

<br>

<br>


## Install Apache
- update local package index of Ubuntu repositories : 
```
sudo apt update

```

<br>

<br>

- install apache server :
```

sudo apt install apache2

```

<br>

<br>

- verify apache installation :
```

apache2 -version

```

<br>

<br>

- list ufw application profiles :
```

sudo ufw app list

```

<br>

<br>

![Screenshot 2022-07-04 at 14 00 57](https://user-images.githubusercontent.com/92983658/177189906-56acccd0-c347-4cbe-86b3-bb5463d10a7e.png)

<br>

<br>

- select `Apache Full` from the profiles list:
```

sudo ufw allow 'Apache Full'

```

<br>

<br>

 - verify apache service is running:
```

curl http://localhost:80

```

 <br>

 <br>
 
 ![Screenshot 2022-07-04 at 13 56 21](https://user-images.githubusercontent.com/92983658/177190000-9fe2eb44-382b-4e03-85ac-c5b15a4a2c56.png)

 <br>

 <br>
 
 ```

sudo systemctl status apache2

```

 <br>

 <br>
 
 ![Screenshot 2022-07-04 at 14 07 43](https://user-images.githubusercontent.com/92983658/177190199-19fbbe14-9f1a-4bdf-9f1f-584499bd10ba.png)

<br>

<br>

- check public domain with AWS EC2 instance public IP address

![Screenshot 2022-07-04 at 14 03 35](https://user-images.githubusercontent.com/92983658/177190361-c86eb711-69fc-4444-bef0-c989bd9aecb4.png)

<br>

<br>

## Install MYSQL on AWS EC2 Ubuntu from SSH
- on SSH terminal:
```

sudo apt-get install mysql-server
sudo mysql

``` 

<br>

<br>

![Screenshot 2022-07-04 at 14 15 24](https://user-images.githubusercontent.com/92983658/177191148-d0bc9731-85d8-440b-bfb9-acf4154d2ec6.png)

<br>

<br>

- define password:

```

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

```

<br>

<br>

```

sudo mysql_secure_installation

```

<br>

<br>

![Screenshot 2022-07-04 at 14 16 58](https://user-images.githubusercontent.com/92983658/177191815-e880828d-ef19-47c1-8fed-da8d6b2d6aee.png)

<br>

<br>

- test MYSQL console: 
```
sudo mysql -u root -p

```

<br>

<br>

![Screenshot 2022-07-04 at 14 20 44](https://user-images.githubusercontent.com/92983658/177192320-2070770a-a4ea-4b5b-8afc-4a0a9819aa2f.png)

<br>

<br>

- `exit`

<br>

<br>

## Install PHP on EC2 Ubuntu
- install PHP with its common extensions:
```

 sudo apt-get install php libapache2-mod-php php-mysql php-curl php-gd php-json php-zip php-mbstring

```

<br>

<br>

- confirm PHP version:
```

php -v

```

<br>

<br>

![Screenshot 2022-07-04 at 14 30 13](https://user-images.githubusercontent.com/92983658/177193126-140b605c-a344-4936-8598-64a002b898e1.png)

<br>

<br>


## Create Virtual Host for a website using APACHE
- create a new directory `projectlamp` in `/var/www/`:

 ```
 sudo mkdir /var/www/projectlamp

```

<br>

- assign ownership of the directory with the current system user

```

sudo chown -R $USER:$USER /var/www/projectlamp

```

<br>

- create and open a new configuration file in Apache’s sites-available directory:

``` 

sudo nano /etc/apache2/sites-available/projectlamp.conf

```

<br>

<br>

![Screenshot 2022-07-04 at 14 36 29](https://user-images.githubusercontent.com/92983658/177193901-913275bd-1b8d-46d8-8120-35f337112d94.png)

<br>

<br>

- paste configuration and exit with `ctrl - X`

<br>

<br.

![Screenshot 2022-07-04 at 17 45 31](https://user-images.githubusercontent.com/92983658/177194395-6f42cb15-47d7-4099-b496-ed14afd7e61e.png)

<br>

<br>

- enable a new virtual host:

```

sudo a2ensite projectlamp

```

<br>

- check for errors:

```
sudo apache2ctl configtest

```

<br>

<br>
 
- reload Apache to confirm changes:

```

sudo systemctl reload apache2

```

<br>

<br>

- create an index.html file in `/var/www/projectlamp/index.html`:

```
sudo nano /var/www/projectlamp/index.html

```

<br>

<br>

- paste in test content

```
Hello LAMP from hostname <public DNS address here> with public IP <public IP address here>

```

<br>

- `ctrl - X` to exit
- type `y` to save

<br>

<br>

![Screenshot 2022-07-04 at 15 44 20](https://user-images.githubusercontent.com/92983658/177195615-03c4bfba-1fd7-46c6-ad59-be8b460f5aea.png)

<br>

<br>

- check public IP address

<br>

<br>

![Screenshot 2022-07-04 at 15 27 01](https://user-images.githubusercontent.com/92983658/177195815-b212a0e1-924d-4f1d-a06d-98d818c0df25.png)

<br>

<br>

## Enable PHP on website
- configure `dir.conf` file:

```
sudo nano /etc/apache2/mods-enabled/dir.conf

```

<br>

<br>

- move PHP index file `index.php` to the first position after `Directory Index`

<br>

<br>

![Screenshot 2022-07-04 at 15 45 26](https://user-images.githubusercontent.com/92983658/177196467-ef74912b-2f81-4a0c-b241-c12a6faba7b0.png)

<br>

<br>

*test PHP installation:*
- create `index.php` in root folder:

```

nano /var/www/projectlamp/index.php

```

<br>

<br>

- paste the following:

```

<?php
phpinfo();

```

<br>

<br>

- exit and save
- refresh the webpage

<br>

<br>

![Screenshot 2022-07-04 at 15 48 49](https://user-images.githubusercontent.com/92983658/177197231-5cd13ca8-9b17-482d-8cc9-dbc6683a52e0.png)

<br>

<br>


- delete the index.php file once finished checking  PHP details

```

sudo rm /var/www/projectlamp/index.php

```

<br>

<br>

- restart Apache webserver

```

sudo service apache2 restart

```

<br>

<br>
