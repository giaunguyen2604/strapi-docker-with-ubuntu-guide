## STEPS TO RUN A STRAPI APP WITH PRODUCTION UBUNTU SERVER  

Table of contents:
1. [Connect to Server](#connect)
2. [Add Swap Space](#swapspace)
3. [Install & Setup Environment](#environment)
    - [Docker](#docker)
    - [Nginx](#nginx)
4. [Install DBMS (Mysql)](#dbms)
   - [Import *.sql file](#importsqlfile)
5. [Config Strapi App](#cfstrapi)
6. [Config Nginx](#cfnginx)
7. [Add SSL](#ssl)
### 1. Connect to Server <a name='connect'></a>
- Generate key pair (public/private)
- Open folder ~/.ssh create a config file with name is `config`
- Add information in that file:
    ```javascript
      Host te-aws-website-api
      	HostName 13.228.46.14
      	User ubuntu
      	IdentityFile ~/.ssh/privatekeyfilename
    ```
	to config file.
- Open terminal typing `ssh te-aws-website-api` to connect to server.


### 2. Add Swap Space <a name='swapspace'></a>
Source: [Click to go to document](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-16-04)

Include 6 steps:

**1. Check the System for Swap Information**
```javascript
  sudo swapon --show
```
- You can verify that there is no active swap using the free utility: `free -h`
- Check Available Space on the Hard Drive Partition: `df -h`

**2. Create a Swap File**
```js
sudo fallocate -l 1G /swapfile
```
- Verify that the correct amount of space was reserved: `ls -lh /swapfile`
  
**3. Enabling the Swap File**
- Make the file only accessible to root by typing: 
  ```js
  sudo chmod 600 /swapfile
  ```
- Verify the permissions change by typing:
  ```js
  ls -lh /swapfile
  ```
As you can see, only the root user has the read and write flags enabled.
- Mark the file as swap space by typing:
  ```js
  sudo mkswap /swapfile
  ```
- Enable the swap file, allowing our system to start utilizing it:
  ```js
  sudo swapon /swapfile
  ```
- Verify that the swap is available by typing:
  ```js
  sudo swapon --show
  ```
- Check the output of the free utility again: `free -h`
  
**4. Make the Swap File Permanent**
- Back up the /etc/fstab file in case anything goes wrong:
  ```js
  sudo cp /etc/fstab /etc/fstab.bak
  ```
- Add the swap file information to the end of your /etc/fstab file:
  ```js
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  ```
**5. Adjusting the Swappiness Property**
> The swappiness parameter configures how often your system swaps data out of RAM to the swap space. This is a value between 0 and 100 that represents a percentage.
- See the current swappiness value:
  ```js
  cat /proc/sys/vm/swappiness
  ```
- For a Desktop, a swappiness setting of 60 is not a bad value. For instance, to set the swappiness to 10:
  ```js
  sudo sysctl vm.swappiness=10
  ```
- Set this value automatically at restart by adding the line to our `/etc/sysctl.conf` file:
  ```js
  sudo nano /etc/sysctl.conf
  ```
  At the bottom, you can add:
  ```js
  vm.swappiness=10
  ``` 
  Save and close the file

**6. Adjusting the Cache Pressure Setting**
> This setting configures how much the system will choose to cache inode and dentry information over other data.

- The current value by querying the proc filesystem:
  ```js
  cat /proc/sys/vm/vfs_cache_pressure
  ```
- We can set this to a more conservative setting like 50:
  ```js
  sudo sysctl vm.vfs_cache_pressure=50
  ```
- Same as swappiness setting, we can add it to our configuration file:
  ```js
  sudo nano /etc/sysctl.conf
  ```
  At the bottom, add the line:
  ```js
  vm.vfs_cache_pressure=50
  ```
  Save and close the file when you are finished
### 3. Install & Setup Environment <a name='environment'></a>
#### 3.1. Install Docker <a name='docker'></a>
Source: [Click to go to document](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
- First, update your existing list of packages:
  ```js
  sudo apt update
  ```
- Install a few prerequisite packages which let apt use packages over HTTPS:
  ```js
  sudo apt install apt-transport-https ca-certificates curl software-properties-common
  ```
- Add the GPG key for the official Docker repository to your system:
  ```js
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  ```
- Add the Docker repository to APT sources:
  ```js
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
  ```
- Update the package database with the Docker packages:
  ```js
  sudo apt update
  ```
- Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:
  ```js
  apt-cache policy docker-ce
  ```
- `docker-ce` is not installed, but the candidate for installation is from the Docker repository for Ubuntu
- Finally, install Docker:
  ```js
  sudo apt install docker-ce
  ```
- Check that it’s running:
  ```js
  sudo systemctl status docker
  ```
##### Executing the Docker Command Without Sudo
If you want to avoid typing sudo whenever you run the docker command:
- Add your username to the docker group:
  ```js
  sudo usermod -aG docker ${USER}
  ```
- To apply the new group membership, type the following:
  ```js
  su - ${USER}
  ```
- Confirm that your user is now added to the docker group:
  ```js
  id -nG
  ```
#### 3.2. Install Nginx <a name='nginx'></a>
Source: [Click to go to document](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)

**1. Install Nginx**
```JS
sudo apt update
sudo apt install nginx
```
**2. Checking your Web Server**
- Check with the systemd init system to make sure the service is running:
  ```js
  systemctl status nginx
  ```
- ```js
  ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
  ```
- Enter it into your browser’s address bar:
  ```js
  http://your_server_ip
  ```
**3. Setting Up Server Blocks (Recommended)**
- Create the directory for `example.com` as follows, using the `-p` flag to create any necessary parent directories:
  ```js
  sudo mkdir -p /var/www/example.com/html
  ```
- Assign ownership of the directory with the `$USER` environment variable:
  ```js
  sudo chown -R $USER:$USER /var/www/example.com/html
  ```
- ```js
  sudo chmod -R 755 /var/www/example.com
  ```
- Create a sample `index.html` page using nano or your favorite editor:
  ```js
  nano /var/www/example.com/html/index.html
  ```
- Inside, add the sample HTML (such as print Hello world) &#8594; Save and close the file when you are finished.
- Let’s make a new file at `/etc/nginx/sites-available/example.com`
- Paste in the following configuration block:
  ```js
  server {
        listen 80;
        listen [::]:80;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
  }
  ```
- Creating a link from it to the sites-enabled directory, which Nginx reads from during startup:
  ```js
  sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
  ```
- To avoid a possible hash bucket memory problem. Open the file:
  ```js
  sudo nano /etc/nginx/nginx.conf
  ```
- Find the `server_names_hash_bucket_size` directive and remove the **#** symbol &#8594; save and close file.
- Test your Nginx files: 
  ```js
  sudo nginx -t
  ```
- If there aren’t any problems, restart Nginx to enable your changes:
  ```js
  sudo systemctl restart nginx
  ```
### 4. Install DBMS (Mysql) with Docker <a name='dbms'></a>
- Install Mysql image: 
  ```javascript
  docker run -p 3306:3306 -d --name mysql -e MYSQL_ROOT_PASSWORD=password mysql/mysql-server
  ```

  Note: 3306:3306 &#8594; port local : port docker

- Verify with command: `docker ps` (list all Images)
- Login to MySQL within the docker container using the docker exec command: 
  ```javascript
  docker exec -it mysql bash
  ```
- Access mysql terminal: 
  ```javascript
  mysql -uroot -ppassword
  ```
- You can create user & grant privileges:
  ```javascript 
  CREATE USER ‘nameuser’@‘%’ IDENTIFIED BY 'password'; 
  ```
  ```javascript
  GRANT ALL PRIVILEGES ON * . * TO ‘nameuser’@‘%’;
  ```
- You can check list users with cmd: `SELECT host, user FROM mysql.user;`
- Create a database with command: `CREATE DATABASE testDB;`

**Copying a sql file to docker container for import**<a name='importsqlfile'></a>
Source: [Guide](https://grant-bartlett.com/blog/copying-a-sql-file-to-docker-container-for-import/)
- First find the docker container:
  ```js
  docker ps
  ```
- This will list out a bunch of container ids. Find the one relating to mysql.
  ```js
  CONTAINER ID        IMAGE                   ...
  [YourContainerId]        mysql:5.7.29       ...CONTAINER ID        IMAGE                   
  ```
- Now copy the sql file using `docker cp` into our container.
  ```js
  docker cp your.sql [YourContainerId]:/your.sql
  ```
- Once complete, we need to login to our container and import to our mysql database.
  ```js
  docker exec -it [YourContainerId/name] bash
  ```
- Once we're logged in, we can now login to mysql.
  ```js
  mysql -u root -p 
  ```
  After that, enter the password
- To see your databases, run `show databases;`.
- Select the database you're going to be importing to:
  ```js
  use your-database-name
  ```
- Now import by running:
  ```js
  source your.sql
  ```
- Done
### 5. Config Strapi App <a name='cfstrapi'></a>
- In `config` folder of Strapi App source, , modify the `database.js` file with fields such as `client`, `host`, `post`, `database`, `username`, `password` which match with the information created at step 4.
- You must install package `mysql2` before: `yarn add mysql2`.
 Here is an example:
 ```javascript
 module.exports = ({ env }) => ({
  defaultConnection: 'default',
  connections: {
    default: {
      connector: 'bookshelf',
      settings: {
        client: 'mysql2',
        host: 'localhost',
        port: 3306,
        database: ‘namedatabase’,
        username: ‘nameuser’,
        password: 'password',
      },
      options: {},
    }
  },
});
 ```
 - You need a process management such as PM2 to keep your app running 24/7. To install typing `yarn global add pm2`
 - Command to PM2 working with scripts in `package.json`: 
 ```js
 pm2 start yarn --interpreter bash --name nameprocess -- start
 ```
 &#8594; `-- start is a script in package.json`
### 6. Config Nginx <a name='cfnginx'></a>
Source: [Document in Strapi website](https://strapi.io/documentation/developer-docs/latest/setup-deployment-guides/deployment/optional-software/nginx-proxy.html)
In `/etc/nginx/sites-available/strapi.conf`:
```js
server {
    # Listen HTTP
    listen 80;
    server_name {your_servername || IP Address};

     # Proxy Config
    location / {
        proxy_pass http://{id_address}:{port};
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_pass_request_headers on;
    }
}
```
- Creating a link from it to the sites-enabled directory:
```js
sudo ln -s /etc/nginx/sites-available/strapi.conf /etc/nginx/sites-enabled/
```
### 7. Add SSL <a name='ssl'></a>
- Follow the steps in the document: [Link document](https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx)


