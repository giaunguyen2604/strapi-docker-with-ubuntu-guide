## STEPS TO RUN A STRAPI APP WITH PRODUCTION UBUNTU SERVER  

Table of contents:
1. [Connect to Server](#connect)
2. [Add Swap Space](#swapspace)
3. [Install & Setup Environment](#environment)
    - [Docker](#docker)
    - [Nginx](#nginx)
4. [Install DBMS (Mysql)](#dbms)
5. [Config Strapi App](#cfstrapi)
6. [Config Nginx](#cfnginx)
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

#### 3.2. Install Nginx <a name='nginx'></a>
Source: [Click to go to document](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)

### 4. Install DBMS (Mysql) with Docker <a name='dbms'></a>
- Install Mysql image: 
  ```javascript
  docker run -p 3306:3306 -d --name mysql -e MYSQL_ROOT_PASSWORD=password mysql/mysql-server
  ```

  `Note: 3306:3306 &#8594; port local : port docker`

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
- Create a database with command: `CREATE DATABASE testDB;`
### 5. Config Strapi App <a name='cfstrapi'></a>
- In `config` folder of Strapi App source, , modify the `database.js` file with fields such as `client`, `host`, `post`, `database`, `username`, `password` which match with the information created at step 4.
 
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
You must install package `mysql2` before: `yarn ad mysql2`.
### 6. Config Nginx <a name='cfnginx'></a>

