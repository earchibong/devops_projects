# Client-Server Architecture In MYSQL
MySQL, being a relational database management system, is designed to work in a client-server architecture by default. The server component of MySQL, commonly known as the MySQL server or MySQL daemon, runs as a separate process and listens for client connections on a specific network port (typically port 3306). Clients, which can be applications or other systems, connect to the MySQL server to send queries and retrieve data.

<br>

<br>

## Project Steps
- <a href="https://github.com/earchibong/devops_projects/blob/main/client_server_msql.md#step-one-create-and-configure-2-ubuntu-virtual-servers">Create and configure 2 Ubuntu Virtual Servers</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/client_server_msql.md#step-two-install-mysql-server-software-on-mysql_server-instance">Install Mysql Server Software on mysql_server instance</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/client_server_msql.md#step-three-configure-mysql">Configure Mysql Server</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/client_server_msql.md#step-four-create-dedicate-user--grant-privileges">Create Dedicate User & Grant Privileges</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/client_server_msql.md#step-five-test-mysql-server">Test Mysql-server</a>
- <a href=" ">Install Mysql-Client Software on Client Server</a>
- <a href=" ">Configure MySQL server to accept connections</a>
- <a href=" ">Connect to MySQL server from MySQL client</a>

<br>

<br>

## Requirements:
- 2 ubuntu instances(one as a server, the other as a client)
AWS EC2 instances will be used for this architecture.

<br>

<br>

## Step One: Create And Configure 2 Ubuntu Virtual Servers
- name the first server: mysql_server
- name the second server: mysql_client

<br>

<br>

## Step Two: Install Mysql Server Software on mysql_server instance
- update the package index of the server:
- install MYSQL server package:
- ensure the server is running using the `systemctl start` command:

<br>

```

sudo apt update
sudo apt install mysql-server -y
sudo systemctl start mysql.service


```

<br>

<br>



## Step Three: Configure Mysql
- Adjust how **root** Mysql user authenticates
   - open Mysql prompt: `sudo mysql`
   - run the following `ALTER USER` command to change the root user’s authentication method to one that uses a password

<br>

  ```
   
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
   
   ```
   *the above changes the authentication method to `mysql_native_password`*
  
<br>

<br>

   - exit Mysql prompt: `exit`

   <br>
   
   ![ALTER_USER](https://user-images.githubusercontent.com/92983658/180238039-665016a1-1e42-411d-8451-90cb69a437b9.png)

   <br>

   <br>

   
- Run Mysql security script: 
- authenticate as the root MySQL user using a password : 
- go back to using the default authentication method:

<br>


 ```

sudo mysql_secure_installation
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;


 ```
 *the above command allows to connect to MySQL as root once again user using the `sudo mysql` command.*
 

 <br>

 <br>
 
 ![ROOT_USER](https://user-images.githubusercontent.com/92983658/180242415-270afcef-9809-4d4f-9d8b-4724059c4e63.png)
 
 <br>

 <br>
 
## Step Four: Create Dedicate User & Grant Privileges

- invoke mysql with `sudo` privileges to gain access to the root MySQL user: 
- create a new user with a CREATE USER statement:

<br>

```

sudo mysql
CREATE USER 'username'@'host' IDENTIFIED WITH mysql_native_password BY 'password';

example:
CREATE USER 'project_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

```

<br>

<br>

- create a database named `testDB`:
- Grant pivileges to user on the database we created: 

<br>

```

CREATE DATABASE testDB;
GRANT ALL PRIVILEGES ON testDB.* TO 'project_user'@'%';

# note: The general syntax for granting user privileges is as follows:
# GRANT PRIVILEGE ON database.table TO 'username'@'host';
# GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *DB* TO 'username'@'localhost' WITH GRANT OPTION;

#to grant all privileges use the following:
GRANT ALL PRIVILEGES ON *.* TO 'user'@'localhost' WITH GRANT OPTION;


```

<br>

<br>

![PRIVILEGES](https://user-images.githubusercontent.com/92983658/180245714-244253ca-9eb3-46e0-b08b-eb7af461754c.png)


<br>

<br>

- free up memory with the `flus privilege` command: `FLUSH PRIVILEGES;`
- exit Mysql prompt: `exit`

**note:In the future, to log in as your new MySQL user, you’d use a command like the following:** : `mysql -u user -p`
*The -p flag will cause the MySQL client to prompt you for your MySQL user’s password in order to authenticate.*

<br>

<br>

## Step Five: Test Mysql-server
- check mysql status:

```

systemctl status mysql.service


```

<br>

<br>

![STATUS](https://user-images.githubusercontent.com/92983658/180248102-5108294d-46b6-425e-b921-38dd13261972.png)

<br>

<br>

**note: If MySQL isn’t running, you can start it with `sudo systemctl start mysql`**

<br>

<br>

## Step Six: Install Mysql-Client Software on Client Server
- update the package index of the server: `sudo apt update`
- install MYSQL client package: `sudo apt install mysql-client -y`
- update inbound security rules on `mysql_server`:
  - under connection type select: `MySQL/Aurora`
  - for IP Address use: `enter mysql_client server's local IP address/32`
<br>

<br>

 ![INBOUND](https://user-images.githubusercontent.com/92983658/180257573-d83e8b0a-a7ea-436f-bb4f-60440eedfb98.png)

<br>

<br>


## Step Seven: Configure MySQL server to accept connections
- On `mysql_server` edit connection settings: `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
- Search for `bind-address` and replace `127.0.0.1` with `0.0.0.0` save and exit

<br>

<br>

![BIND](https://user-images.githubusercontent.com/92983658/180254888-034d6d99-02d0-420c-89d9-880694cbd889.png)

<br>

<br>

- restart Mysql prompt: `sudo systemctl restart mysql`


<br>

<br>

## Step Eight: Connect to MySQL server from MySQL client

- in `mysql-client` connect to `mysql_server` with this command: `sudo mysql -u project_user -p -h mysql-server local ip`
- Enter mysql server password when prompted
- check connection by running the following command: `SHOW DATABASES;`

<br>

<br>

![DATABASE_CONNECT](https://user-images.githubusercontent.com/92983658/180263064-bb0a6ae6-9236-4390-8a2a-24340b05897d.png)

<br>

*If you get an output similar to the above image, then a fully functional MySQL Client-Server set up has been deployed successfully.*

<br>



