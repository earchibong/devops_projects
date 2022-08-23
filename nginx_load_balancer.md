# Horizontal Scalability With NginX And SSL/TLS Encryption

This project consists of two parts:
- Configure Nginx as a Load Balancer
- Register a new domain name and configure secured connection using SSL/TLS certificates

The target architecture will look like this:

![nginx_lb_architecture](https://user-images.githubusercontent.com/92983658/186160254-c8319f4d-736d-431b-a5ab-c97982a4d6c8.png)

## Part One: Configure NginX as a Load Balancer
- Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it `Nginx LB` 
- open TCP port 80 for HTTP connections, also open TCP port 443 (this port is used for secured HTTPS connections)
- Update the instance and Install Nginx: 

```

sudo apt update
sudo apt install nginx

```

- Update /etc/hosts file for local DNS with Web Servers’ names  and their local IP addresses

```

#Open this file on the LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2

```

![etc-hosts-update](https://user-images.githubusercontent.com/92983658/186168944-217ce448-756c-4c54-b958-da121a4ea6a6.png)


- update NginX Load Balancer config file with  Web Servers’ names defined in `/etc/hosts`: `sudo vi /etc/nginx/nginx.conf`

```

#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;

```

![nginx_config_update](https://user-images.githubusercontent.com/92983658/186170657-9f3eb4b0-a94b-4d2d-a13b-19f307df155d.png)


- Restart Nginx and make sure the service is up and running
```

sudo systemctl restart nginx
sudo systemctl status nginx

```

![nginx_status](https://user-images.githubusercontent.com/92983658/186171043-b87c0236-533d-42fa-8345-99a4c6197bb6.png)


## Part Two: Register A New Domain And Configure Secured Connection Using SSL/TLS Certificates
- Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
- 
