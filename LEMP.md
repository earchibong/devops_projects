#LEMP INSTALLATION ON AWS
## STEP ONE: Create and configure virtual server on AWS
- <a href="https://github.com/earchibong/devops_training/blob/main/LAMP.md">See here</a>

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
## STEP FOUR: Install PHP
## STEP FIVE: Configure Nginx
## STEP SIX: Test Nginx and PHP
## STEP SEVEN: Retrieve data from database with PHP
