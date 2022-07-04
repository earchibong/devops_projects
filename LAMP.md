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
- select `Apache Full` from profiles list:
 `sudo ufw allow 'Apache Full'`
 
 - verify apache service is running:
 `sudo systemctl status apache2`

## STEP THREE: Install MYSQL on AWS EC2 Ubuntu from SSH

## STEP FOUR: Install PHP on EC2 Ubuntu
## STEP FIVE: Create Virtual Host
## STEP SIX: Restart Apache Web ServerServer
