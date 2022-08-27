# Load Balancer With NginX And SSL/TLS Encryption

This project consists of two parts:
- Configure Nginx as a Load Balancer
- Register a new domain name and configure secured connection using SSL/TLS certificates

The target architecture will look like this:

![nginx_lb_architecture](https://user-images.githubusercontent.com/92983658/186160254-c8319f4d-736d-431b-a5ab-c97982a4d6c8.png)

## Part One: Register And Configure New Domain
- Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
- Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it `Nginx LB` 
 - open TCP port 80 for HTTP connections, also open TCP port 443 (this port is used for secured HTTPS connections)

- ### Connect new domain with Nginx Public IP:
- if hosted hoze doesn't exist for new domain on aws route 53, create a new hosted zone:

- **Create hosted zone:**
  - in AWS Route 53 menu, click on `hosted zones`
  - select new domain and click on `create hosted zone`
   - Enter the name of the domain you just purchased, a description, and whether you want the domain to be publicly accessible or private to your internal network.
   - Click on `CREATE HOSTED ZONE` button to finish the configuration.  

- **configure DNS RECORDS:**
  - in AWS Route 53 menu, click on `hosted zones`
  - select new domain and click on domain name
   - Click the `CREATE RECORD` button to get started:
   - Enter in your `A record` information and ensure "A" is selected in the Record Type field.
   - Enter the `NginX Public IP address` into the value.
   - Click the CREATE RECORDS button once you have finished.

 ![A-record_nginx](https://user-images.githubusercontent.com/92983658/187036964-65ba6a2a-f7b0-47e7-b907-5c786952dc57.png)

 
  - create another `A record`:
   - Enter `www` in the `record name`
   - for `record type` and `value` enter the same information as above .
 
 note: more on elastic ips <a href="https://aws.amazon.com/getting-started/hands-on/get-a-domain/">here</a> and <a href="https://medium.com/progress-on-ios-development/connecting-an-ec2-instance-with-a-godaddy-domain-e74ff190c233">here</a>



## Part Two: Configure NginX Server as a Load Balancer

- SSH into EC2 instance
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
    server_name www.your_new_domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;

```

![config_nginx](https://user-images.githubusercontent.com/92983658/186703987-07dd1ad4-86b0-4481-8cdc-54be16afb2b6.png)


- Restart Nginx and make sure the service is up and running
```

sudo systemctl restart nginx
  

```

![nginx_status](https://user-images.githubusercontent.com/92983658/186171043-b87c0236-533d-42fa-8345-99a4c6197bb6.png)


- Test access to webserver using `Http` protocol: `http://you-new-domain-name.com`


![http_test](https://user-images.githubusercontent.com/92983658/187036098-fc205cde-405b-4356-ba22-475d03af1895.png)



- **Install certbot and request for an SSL/TLS certificate**
 - Make sure snapd service is active and running : `sudo systemctl status snapd` 
 - install cerbot: `sudo snap install --classic certbot`
 - Request your certificate:
 ```
 
 sudo ln -s /snap/bin/certbot /usr/bin/certbot
 sudo certbot --nginx

```

![cert](https://user-images.githubusercontent.com/92983658/186872705-c3dec8a4-babd-4315-a329-b32325c4d33c.png)


- Test secured access to your Web Solution by trying to reach `https://<your-domain-name.com>`

 ![https_test](https://user-images.githubusercontent.com/92983658/187036461-c87a0e2b-e8e9-47fb-aca6-cb17bc08fb63.png)

 
 
- **Set up periodical renewal of your SSL/TLS certificate:**
 - You can test renewal command in dry-run mode: `sudo certbot renew --dry-run`
 
![certbot_renew_dryrun](https://user-images.githubusercontent.com/92983658/186874355-d8db9e29-c59d-43f5-af7e-09799e4e4d95.png)

 - schedule job that to runs renew command periodically: 
  - edit the crontab file: `crontab -e`
  - add the following line: `* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`
 
 ![cron_job](https://user-images.githubusercontent.com/92983658/187036583-3f55e346-1ad5-4537-91cd-6b70f82b55b8.png)


## Assign an Elastic IP to Nginx LB server: 
 - In the top search bar of AWS console, enter "Elastic IP". The search bar will return several results on services and features available.
 - Click the ELASTIC IPs - EC2 feature from the list:
   - Once on the Elastic IP addresses screen, click ALLOCATE ELASTIC IP ADDRESS on the top right of the page.
   - Enter details on allocation page and allocate elastic IP
 - Under the Actions menu of `elastic ip addresses` click `associate elastic ip address`:
   - on the asocialte elastic ip address screen, assign the elastic ip to the Nginx LB server

![elastic_nginx_association](https://user-images.githubusercontent.com/92983658/186363962-0491a62c-84ea-465e-8ed6-d57deadd406d.png)

- **configure DNS RECORDS:**
  - in AWS Route 53 menu, click on `hosted zones`
  - select new domain and click on domain name
   - select an `A record` from the list and click `EDIT RECORD` button to get started:
   - Enter the `Elastic IP address` into the value.
   - Click the `SAVE` button once you have finished.

 ![A_record](https://user-images.githubusercontent.com/92983658/186440451-f0ca251e-8c3a-4086-80ec-10ecba84731d.png)
 
  - repeat for the `www` `A record`:
  
 
 note: more on elastic ips <a href="https://aws.amazon.com/getting-started/hands-on/get-a-domain/">here</a> and <a href="https://medium.com/progress-on-ios-development/connecting-an-ec2-instance-with-a-godaddy-domain-e74ff190c233">here</a>


- test access to web solution with Elastic IP: `https://your-domain.com`

![elastic_ip](https://user-images.githubusercontent.com/92983658/187037524-8d2bbba2-d686-4001-b298-5ae03b93a406.png)




