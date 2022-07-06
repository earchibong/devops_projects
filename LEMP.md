#LEMP INSTALLATION ON AWS
## STEP ONE: Create and configure virtual server on AWS

<a href="https://github.com/earchibong/devops_training/blob/main/LAMP.md">See here</a>

## STEP TWO: Install Nginx Server
- update server: `sudo apt update`
- install nginx: `sudo apt install nginx`

![install_nginx](https://user-images.githubusercontent.com/92983658/177517763-5595f1f4-d2b0-423c-badc-a08209aa12c6.png)

- check nginx status: `sudo systemctl status nginx`

![nginx_status](https://user-images.githubusercontent.com/92983658/177517974-ca324d51-1393-4841-b7c6-781976a3da7b.png)

- test local access: `curl http://localhost:80`

![curl_nginx](https://user-images.githubusercontent.com/92983658/177518287-a3392188-3e4f-4017-aaee-66cf5be7e8f5.png)

- check browser: `http://<EC2 IP>` `http://13.40.82.74`

![ip_adress_nginx](https://user-images.githubusercontent.com/92983658/177519074-db7f6595-6155-42ed-be6d-be5fdb5c088e.png)


## STEP THEE: Install Mysql Database
- acquire and install mysql database : `sudo apt install mysql-server`

![mysql_install](https://user-images.githubusercontent.com/92983658/177556240-e7d06a72-1610-47a5-8eb7-7382d446fb76.png)

- log into MYSQL : `sudo mysql`
- set password for root user: `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

![mysql_set_password](https://user-images.githubusercontent.com/92983658/177556904-64fb41c4-83c9-4b27-b8a0-fd212d3ab59d.png)

- exit mysql shell: exit
-  run security script : `sudo mysql_secure_installation`

![musql_security_script](https://user-images.githubusercontent.com/92983658/177557473-71f577bf-95a5-4105-b75e-10f88b72897d.png)

- test mysql installation : `sudo mysql -p`

![mysql_status](https://user-images.githubusercontent.com/92983658/177557854-dd515c11-b290-4aec-a267-80a496062559.png)

- exit mysql : `exit`

## STEP FOUR: Install PHP
- install `PHP fastCGI process manager` and `php-mysql` : `sudo apt install php-fpm php-mysql`

![php_myql_fpm](https://user-images.githubusercontent.com/92983658/177559045-dfb1578c-fbd0-4874-b8aa-461e0c52e15b.png)

## STEP FIVE: Configure Nginx

- create root web directory : `sudo mkdir /var/www/projectLEMP`
- assign ownership of directory: `sudo chown -R $USER:$USER /var/www/projectLEMP`
- open new configuration file: `sudo nano /etc/nginx/sites-available/projectLEMP`
- paste configuration

![nano_php_configuration](https://user-images.githubusercontent.com/92983658/177560662-0fe20e44-4f06-4f4b-ab96-b8cf39f633f6.png)


- save and close file: `ctrl-x`, then `y` then `enter`
- activate configuration: `sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`
- test configuration for syntax errors: `sudo nginx -t`

![activation_configuration](https://user-images.githubusercontent.com/92983658/177561077-06f4179e-004e-47de-8537-5a4427c39eea.png)

- unlink default nginx host: `sudo unlink /etc/nginx/sites-enabled/default`
- reload nginx: `sudo systemctl reload nginx`

- create new index file and test server block: 
`sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html`

- check browser using IP address: `http://ec2 public IP`

![public_IP_test](https://user-images.githubusercontent.com/92983658/177563188-6863e3d1-30c7-42c5-bd1f-a1fbc5335441.png)

## STEP SIX: Test Nginx and PHP
## STEP SEVEN: Retrieve data from database with PHP
