# LEMP INSTALLATION ON AWS

LEMP stack is a popular software stack used for web application development and hosting. It consists of four main components:

**Linux:** The "L" in LEMP represents the Linux operating system. Linux provides a stable and secure foundation for hosting web applications. It is commonly used in combination with other components of the LEMP stack.

**Nginx:** The "N" in LEMP stands for Nginx, which is a high-performance web server and reverse proxy server. Nginx is known for its scalability, speed, and efficiency in handling concurrent connections. It is often used as the front-end web server in the LEMP stack.

**MySQL/MariaDB:** The "M" in LEMP refers to MySQL or MariaDB, which are popular relational database management systems (RDBMS). They provide a robust and scalable solution for storing and managing data required by web applications. MySQL and MariaDB are widely used in conjunction with PHP and other programming languages.

**PHP:** The "P" in LEMP represents PHP, which is a widely used server-side scripting language. PHP allows developers to build dynamic web applications by embedding code within HTML. It provides extensive libraries and frameworks for rapid web development. PHP is typically used in conjunction with Nginx and MySQL/MariaDB in the LEMP stack.

When combined, the LEMP stack provides a complete environment for hosting web applications. Nginx handles the incoming web requests and acts as a reverse proxy to forward requests to the appropriate PHP scripts. PHP processes the requests, generates dynamic content, and communicates with the MySQL/MariaDB database to retrieve or store data.

The LEMP stack is known for its performance, scalability, and compatibility with a wide range of web applications. It is particularly popular for hosting content management systems (CMS) like WordPress, Drupal, and Joomla.

It's worth noting that alternative components can also be used in the LEMP stack, such as PostgreSQL instead of MySQL/MariaDB, or other scripting languages like Python or Ruby instead of PHP. The stack can be customized based on specific application requirements and preferences.

<br>

<br>

## Project Steps
- <a href="https://github.com/earchibong/devops_projects/blob/main/LEMP.md#create-and-configure-a-virtual-server-on-aws">Create and configure virtual server on AWS</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/LEMP.md#install-nginx-server">Install Nginx</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/LEMP.md#install-mysql-database">Install Mysql Database</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/LEMP.md#install-php">Install PHP</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/LEMP.md#configure-nginx">Configure Niginx</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/LEMP.md#test-nginx-and-php">Test Nginx & PHP</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/LEMP.md#retrieve-data-from-the-database-with-php">Retrieve Data From Database With PHP</a>



<br>

<br>

## Create and configure a virtual server on AWS

Get instructions on how to get started with EC@ instances <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html">here</a>
<br>

<br>

## Install Nginx Server
- update server and install nginx:
```
sudo apt update
sudo apt install nginx

```

<br>

<br>

![install_nginx](https://user-images.githubusercontent.com/92983658/177517763-5595f1f4-d2b0-423c-badc-a08209aa12c6.png)

<br>

<br>

- check nginx status:

```

sudo systemctl status nginx

```

<br>

<br>

![nginx_status](https://user-images.githubusercontent.com/92983658/177517974-ca324d51-1393-4841-b7c6-781976a3da7b.png)

<br>

<br>

- test local access:
```

curl http://localhost:80

```

<br>

<br>

![curl_nginx](https://user-images.githubusercontent.com/92983658/177518287-a3392188-3e4f-4017-aaee-66cf5be7e8f5.png)

<br>

<br>

- check browser:

```

http://<EC2 IP>
http://13.40.82.74 #example

```

<br>

<br>

![ip_adress_nginx](https://user-images.githubusercontent.com/92983658/177519074-db7f6595-6155-42ed-be6d-be5fdb5c088e.png)


<br>

<br>

## Install Mysql Database
- acquire and install mysql database :

```
sudo apt install mysql-server

```

<br>

<br>

![mysql_install](https://user-images.githubusercontent.com/92983658/177556240-e7d06a72-1610-47a5-8eb7-7382d446fb76.png)

<br>

<br>

- log into MYSQL and set password for root user :

```

sudo MySQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';


```

<br>

<br>

![mysql_set_password](https://user-images.githubusercontent.com/92983658/177556904-64fb41c4-83c9-4b27-b8a0-fd212d3ab59d.png)

- exit mysql shell: exit
-  run security script :

```

sudo mysql_secure_installation

```

<br>

<br>

![musql_security_script](https://user-images.githubusercontent.com/92983658/177557473-71f577bf-95a5-4105-b75e-10f88b72897d.png)

<br>

<br>

- test mysql installation :

```

sudo mysql -p

```

<br>

<br>

![mysql_status](https://user-images.githubusercontent.com/92983658/177557854-dd515c11-b290-4aec-a267-80a496062559.png)

<br>

<br>

- exit mysql : `exit`

<br>

<br>


## Install PHP
- install `PHP fastCGI process manager` and `php-mysql` :
```

sudo apt install php-fpm php-mysql

```

<br>

<br>

![php_myql_fpm](https://user-images.githubusercontent.com/92983658/177559045-dfb1578c-fbd0-4874-b8aa-461e0c52e15b.png)


<br>

<br>

## Configure Nginx

- create root web directory 
- assign ownership of directory: 
- open new configuration file:
- paste configuration

```

sudo mkdir /var/www/projectLEMP
sudo chown -R $USER:$USER /var/www/projectLEMP
sudo nano /etc/nginx/sites-available/projectLEMP

```

<br>

<br>

![nano_php_configuration](https://user-images.githubusercontent.com/92983658/177560662-0fe20e44-4f06-4f4b-ab96-b8cf39f633f6.png)


<br>

<br>

- save and close file: `ctrl-x`, then `y` then `enter`
- activate configuration:
- test configuration for syntax errors:

```

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
sudo nginx -t

```

<br>

<br>

![activation_configuration](https://user-images.githubusercontent.com/92983658/177561077-06f4179e-004e-47de-8537-5a4427c39eea.png)


<br>

<br>

- unlink default nginx host:
- reload nginx: 
- create new index file and test server block: 
- check browser using IP address:

```

sudo unlink /etc/nginx/sites-enabled/default
sudo systemctl reload nginx

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

http://ec2 public IP

```

<br>

<br>

![public_IP_test](https://user-images.githubusercontent.com/92983658/177563188-6863e3d1-30c7-42c5-bd1f-a1fbc5335441.png)

<br>

<br>


## Test Nginx and PHP
- create a test PHP file in the document root:
- paste PHP code:

```

sudo nano /var/www/projectLEMP/info.php
<?php
phpinfo();

```

<br>

<br>

![Screenshot 2022-07-06 at 10 04 30](https://user-images.githubusercontent.com/92983658/177569538-4ce68948-43e4-486e-bbb1-fb9babf92543.png)

<br>

<br>

- write and save file: `ctrl-x then y then enter`
- access new php file in browser with `/info.php` extension: `http://ec2 public ip/info.php`

<br>

<br>

 ![Screenshot 2022-07-06 at 17 11 28](https://user-images.githubusercontent.com/92983658/177596080-4bf0fdcb-57bc-4ce4-9af5-d27f48316555.png)

<br>

<br>

- remove test file `info.php` after checking relevant information: `sudo rm /var/www/your_domain/info.php`

<br>

<br>

## Retrieve data from the database with PHP

- connect to mysql console: 
- create a new database:

```

sudo mysql -p
CREATE DATABASE `example_database`;


```

<br>

<br>

![Screenshot 2022-07-06 at 15 21 12](https://user-images.githubusercontent.com/92983658/177572681-89c46d10-eaea-4c99-a8e3-b4b8429a256b.png)

<br>

<br>

- create a new user: 
- give user permission over database: 
- exit mysql: 
- test user permissions:
- confirm access to database:

```

CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
GRANT ALL ON example_database.* TO 'example_user'@'%';
exit
mysql -u example_user -p
SHOW DATABASES;

```

<br>

<br>

![Screenshot 2022-07-06 at 15 28 19](https://user-images.githubusercontent.com/92983658/177574279-2f2669cd-4c53-417d-96d6-3231fadec1d4.png)

<br>

<br>

- create test table '`to-do list`:
```

CREATE TABLE example_database.todo_list (
  item_id INT AUTO_INCREMENT,
  content VARCHAR(255),
  PRIMARY KEY(item_id)
);

```

<br>

<br>


![Screenshot 2022-07-06 at 16 02 36](https://user-images.githubusercontent.com/92983658/177582132-f6c7a4ff-9918-4659-829e-0c0d0d82110f.png)


<br>

<br>

- insert new rows of content:
```

INSERT INTO
  example_database.todo_list (content)
VALUES
  ("My first important item");

```

 - repeat multiple times with different values

<br>

<br>
  
  ![Screenshot 2022-07-06 at 16 12 02](https://user-images.githubusercontent.com/92983658/177584113-dab94a42-fede-4589-a2db-46c4630afa01.png)

<br>

<br>

- confirm data successfully saved:
```

SELECT * FROM example_database.todo_list;

```

<br>

<br>


![Screenshot 2022-07-06 at 16 13 35](https://user-images.githubusercontent.com/92983658/177584714-251dd25c-d560-428e-9d78-a4af57b3594d.png)

<br>

<br>

- exit mysql: `exit`

- *create PHP script to query MYSQL for database content* : `nano /var/www/projectLEMP/todo_list.php`
- copy php script into `todo-list.php`

<br>

<br>


![Screenshot 2022-07-06 at 16 28 43](https://user-images.githubusercontent.com/92983658/177587746-2f87e6d5-a7a4-471f-ae6c-7cf9cbe502ad.png)


<br>

<br>

- test configuration: 
- restart php-fpm and nginx:
- access todo_list in browser:

```

sudo service nginx configtest
sudo service php7.0-fpm restart` and `sudo service nginx restart
http://ec2 public ip/todo_list.php

```

<br>

<br>

![Screenshot 2022-07-06 at 17 05 47](https://user-images.githubusercontent.com/92983658/177595072-890e9d73-22d1-4780-b5c3-ce9c0ce3b3e5.png)

<br>

<br>

