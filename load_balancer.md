# Project: Deploy And Configure An Apache Load Balancer

Load Balancer to be used with tooling website solution that makes access to DevOps tools within the corporate infrastructure easily 
accessible. Users should be served by Web servers through the Load Balancer.

## Pre-requisite:
In this project the following components will be used for the solution:

- Infrastructure: AWS
- Webserver Linux: 2 Red Hat Enterprise Linux 8
- Database Server: Ubuntu 20.04 + MySQL
- Storage Server: Red Hat Enterprise Linux 8 + NFS Server

## STEP ONE: Configure Load Balancer Server

- launch an ubuntu EC2 instance named `apache-lb`
- open `TCP port 80`
- install apache Laod Balancer and configure it to point to webservers
```

#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2

```

- ensure `apache` is running: `sudo systemctl status apache2`


![apache](https://user-images.githubusercontent.com/92983658/184630107-42c3bfed-0d80-4243-82c6-ba4bfa870732.png)

- ### Configure Load Balancing
```

sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

#Restart apache server

sudo systemctl restart apache2

```

![load_balancer_configure](https://user-images.githubusercontent.com/92983658/184632041-c2ce14cd-b5ac-48fd-8562-08e7dbc04dd1.png)


-*Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.*: 
  - `sudo umount /var/log/httpd`
  - If the NFS mount has an entry in the fstab file, remove it or comment it out.
 
 ![umount](https://user-images.githubusercontent.com/92983658/184634907-7e01aeaf-4574-44f6-9ea8-5e87c199d2ec.png)


- verify the configuration works: access your LB’s public IP address or Public DNS name from your browser
`http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`

![load_balancer_confirm](https://user-images.githubusercontent.com/92983658/185124537-ba834754-6b56-412f-be95-18119375b76c.png)


- Open two ssh/Putty consoles for both Web Servers and run following command: `sudo tail -f /var/log/httpd/access_log`
- Refresh your browser page `http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php` several times
- make sure that both servers receive HTTP GET requests from the load balancer

![get_web1](https://user-images.githubusercontent.com/92983658/185140479-977d8c36-9057-4137-acec-ba79ff8a6264.png)

![get_web2](https://user-images.githubusercontent.com/92983658/185140563-7c1a437e-f442-42ce-8e02-49db583011ad.png)

