# Project-Base-Learning-7
DEVOPS TOOLING WEBSITE SOLUTION

## DEVOPS TOOLING WEBSITE SOLUTION

## PROJECT TASK
- Implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

## BACKGROUND KNOWLEDGE

- In previous project  [Project 6](https://www.darey.io/docs/project-6-step-1/)  I implemented a WordPress based solution that is ready to be filled with content and can be used as a full fledged website or blog. Moving further I will add some more value to my solution so that a member of a DevOps team could utilize a set of DevOps tools that will help the team in day to day activities of managing, developing, testing, deploying and monitoring different projects..

In this project,, I will be introducing the concept of file sharing for multiple servers to share the same web content and also a database for storing data related to the website.

- The tools I want our team to be able to use are well known and widely used by multiple DevOps teams, so I will introduce a single DevOps Tooling Solution that will consist of:

- 1 [Jenkins](https://www.jenkins.io) - free and open source automation server used to build [CI/CD](https://en.wikipedia.org/wiki/CI/CD) pipelines.

- 2 [Kubernetes](https://kubernetes.io) – an open-source container-orchestration system for automating computer.

- 3 [Jfrog Artifactory](https://jfrog.com/artifactory/) – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.

- 4 [Rancher](https://www.rancher.com/products/rancher) – an open source software platform that enables organizations to run and manage [Docker](https://en.wikipedia.org/wiki/Docker_(software)) and Kubernetes in production.

- 5 [Grafana](https://grafana.com) – a multi-platform open source analytics and interactive visualization web application.

- 6 [Prometheus](https://prometheus.io) – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.

- 7 [Kibana](https://www.elastic.co/kibana/) – Kibana is a free and open user interface that lets you visualize your Elasticsearch [Elasticsearch](https://www.elastic.co/elasticsearch/) data and navigate the [Elastic](https://www.elastic.co/elastic-stack/) Stack.

Side Self Study : Read about Network-attached storage (NAS), Storage Area Network (SAN) and related protocols like NFS, (s)FTP, SMB, iSCSI. Explore what Block-level storage is and how it is used by Cloud Service providers, know the difference from Object storage.
On the example of AWS services understand the difference between Block Storage, Object Storage and Network File System.

## Setup and technologies used in Project 7

- As a member of a DevOps team, I implemented a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

- In this project I implemented a solution that consists of following components:

- Infrastructure: AWS
- Webserver Linux: Red Hat Enterprise Linux 9
- Database Server: Ubuntu 20.04 + MySQL
- Storage Server: Red Hat Enterprise Linux 9 + NFS Server
- Programming Language: PHP
- Code Repository: [GitHub](https://github.com/darey-io/tooling)

- On the diagram below we can see a common pattern where several **stateless Web Servers** share a common database and also access the same files using **Network File Sytem (NFS)** as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it looks like a local file system from where they can serve the same files.

This project consists of the following servers:
 - Web server(RHEL)
 - Database server(Ubuntu + MySQL)
 - Storage/File server(RHEL + NFS server)

![7_1](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/0d10cd6d-5ecf-4cbc-a713-095719a85375)

It is important to know what storage solution is suitable for what use cases, for this – you need to answer following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Base on this you will be able to choose the right storage system for your solution.

## STEP 1 – ***PREPARE NFS SERVER***

### 1. - I Spinned up a new EC2 instance with RHEL Linux 9 Operating System.
For Rhel 9 server, i used Rhel 9 server(HVM)
Note: Below can still be used : ami RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2 (ami-035c5dc086849b5de)

###  2. - Based on my LVM experience from **Project 6**, I Configured 3 LVM on the Server.

First, I want to create a partition on the physical disk - xvdf, xvdh, xvdg, then switch to logical volume management.

Use gdisk utility to create a single partition on each of the 3 disks - xvdf, xvdh, xvdg, Install lvm2 package, created physical volume, created a volume group, created logical volume, formated the disk with xfs file system


Use gdisk utility to create a single partition on each of the 3 disks - xvdf, xvdh, xvdg
```
sudo gdisk /dev/xvdf 
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```

Install lvm2 package

```
sudo yum install lvm2 -y
sudo lvmdiskscan
```

created physical volume, created a volume group, created 3 logical volume

```
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
sudo pvs

sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
sudo vgs

sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-logs -L 9G webdata-vg
sudo lvcreate -n lv-opt -L 9G webdata-vg
sudo lvs
```

- Instead of formating the disks as ***ext4***,  I formatted the disk as ***xfs***

```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
lsblk
```

### 3. - I ensured there are 3 **Logical Volumes**. ***lv-opt lv-apps, and lv-logs***

I created mount points on **/mnt** directory for the logical volumes as follow:
- Mount **lv-apps** on **/mnt/apps** – To be used by webservers
- Mount **lv-logs** on **/mnt/logs** – To be used by webserver logs
- Mount **lv-opt** on **/mnt/opt** – To be used by Jenkins server in *Project 8*

Directory created and the LV mounted into their various mount points
```
sudo mkdir -p /mnt/apps
sudo mkdir -p /mnt/logs
sudo mkdir -p /mnt/opt

sudo mount /dev/webdata-vg/lv-apps /mnt/apps
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```

### 4. - I installed NFS server, configured it to start on reboot and made sure it is up and running

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
![7_2](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/19d68401-1eac-4032-9e91-23dcf13fb308)

### 5. -I exported the mounts for webservers’ ***subnet cidr*** to connect as clients. 
For simplicity, I installed all the three Web Servers inside the same subnet, but in production set up I would probably want to separate each tier inside its own subnet for higher level of security.

To check the ***subnet cidr*** – I opened my EC2 details in AWS web console and located ‘Networking’ tab and opened a Subnet link:

![7_3](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/bfa7b0f0-43b6-42e0-8194-59eaff2a3ac4)

- I made sure I set up permission that will allow the Web servers to read, write and execute files on NFS:

Change ownership of the directory

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt
```

Allow read, write , Execute access on these directories, and then Restart the NFS Server

```
sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```

I Configured access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.80.0/20 ):

```
sudo vi /etc/exports

Script
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Script modified & Executed
/mnt/apps 172.31.80.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.80.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.80.0/20(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!
```

I Exported modified file, so that the web servers will be able to access it

`sudo exportfs -arv`


### 6. - I checked which port is used by NFS and opened it using Security Groups (add new Inbound Rule)

- `rpcinfo -p | grep nfs` 

The port used is **2049** 

Also ensure HTTP is opened

![7_4](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/03fc04db-f470-4c67-ab33-65913ff606c9)


## STEP 2 — ***CONFIGURE THE DATABASE SERVER***

### 1. - I installed MySQL server on the Ubuntu OS

USED for installing MYSQL on a Ubuntu OS, Always run APT update 1st
NOTE - Ubuntu Server 20.04 LTS (HVM), SSD Volume Type was used

```
sudo apt update
sudo apt install mysql-server -y
sudo systemctl restart mysql
sudo systemctl enable mysql
sudo systemctl status mysql
```

USED for installing MYSQL on a RedHart OS, Always run YUM update 1st
```
sudo yum update
sudo yum install mysql-server -y
sudo systemctl restart mysql
sudo systemctl enable mysql
sudo systemctl status mysql
```

Remember to install some security on mysql DB while on production network, you may want to run this... `sudo mysql_secure_installation` but we are not going into this. Also note, you installed mysql, not mysqld.

### 2. - I created a database and named it `tooling`

### 3. - I created a database user and named it `webaccess`

### 4. - I granted permission to `webaccess` user on `tooling` database to do anything only from the webservers subnet cidr

```
SCRIPT
sudo mysql
CREATE DATABASE tooling;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit

EXECUTED
sudo mysql
CREATE DATABASE tooling;
CREATE USER `webaccess`@`172.31.80.0/20` IDENTIFIED BY 'password';
GRANT ALL ON tooling.* TO 'webaccess'@'172.31.80.0/20';
FLUSH PRIVILEGES;
SHOW DATABASES;
use tooling;   - try accessing the new tooling DB
show tables;    - u will see an empty table
use mysql;       - try accessing an existing database - mysql
show tables;
describe table proxies_priv;
select 'partition', 'possible_keys', 'key' from proxies_priv
exit;
```
![7_5](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/2cd6341a-05be-46a6-8446-c7ab3bd3c1c9)

### 5. - Allow Port Access, Set Binding Address
I ensured below ports were opened on the DB server, this is to allow the web server connect to mysql on the DB Server
![7_12](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/38e9330b-a9af-4bfc-af2e-68628ca6d4d9)

I also ensured the binding address was set...
Bind-Address = 127.0.0.1 means the DB is only listening for connection from the local host,this is why we could connect to the DB locally, but could connect remotely. Bind address can be restricted to an IP, a subet or it is opened to ALL when 0.0.0.0 is used

sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

![7_13](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/74634906-6815-4bfd-b2e2-f891357f275e)

MYSQL on the DB Server was restarted and its status was checked

```
sudo systemctl restart mysql
sudo systemctl status mysql
```


## Step 3 — ***PREPARE THE WEB SERVER***

- I ensured that the Web Servers can serve the same content from shared storage solutions, in this case – NFS Server and MySQL database.
Knowing that one DB can be accessed for **reads** and **writes** by multiple clients. For storing shared files that the Web Servers will use – I utilized NFS and mounted previously created Logical Volume **lv-apps** to the folder where Apache stores files to be served to the users i.e the **(/var/www)** folder.

- This approach will make the Web Servers **stateless**, which means I will be able to add new ones or remove them whenever I need, and the integrity of the data (in the database and on NFS) will be preserved.

**So basically, i seek to mount on my web server, all the logical volumes which i have created previously on the NFS server

During the next steps I did the following:

 - Configured NFS client (this step must be done on all three servers)

 - Deployed a Tooling application to the Web Servers into a shared NFS folder

 - Configured the Web Servers to work with a single MySQL database

### 1 . - I launched a new EC2 instance with RHEL 9 Operating System

### 2. - I installed NFS client using below command
Note - without this installation, i will not be able to access the NFS Server from the web server

`sudo yum install nfs-utils nfs4-acl-tools -y`

### 3 - I mounted **/var/www/** and target the NFS server’s export for apps

```
sudo mkdir /var/www

Script
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

Executed Script
sudo mount -t nfs -o rw,nosuid 172.31.86.229:/mnt/apps /var/www
```
Note: the /var/www is located locally on our web server, while /mnt/apps is located remotely on our NFS. So we target to connect our `/var/www` directory on our webserver with the `/mnt/apps` on NFS server. This is acheived by mounting the NFS server directory to the webserver directory  From my Web Server, i ran `df -h` cmd to confirm the mount was successful.

### 4 - I verified that NFS was mounted successfully by running `df -h`. I ensured that the changes will persist on Web Server after reboot:

```
df -h
sudo vi /etc/fstab
```

- I added the following line inside the file in the fstab

```
Script
`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

Executed Script
172.31.86.229:/mnt/apps /var/www nfs defaults 0 0
```
So basically, the above has been done to ensure that our mounts remain intact when the server reboots. This is achieved by configuring the fstab directory.

### 5 - I installed  Apache on the Web server
Remi's Repository  [Remi’s repository](http://www.servermom.org/how-to-enable-remi-repo-on-centos-7-6-and-5/2790/). Without the Apache, the web server will not be able to serve content to the web users. Note...Nginx, Apache etc are the popular web servers clients out there.

Note: ensure you run sudo yum update to install all the dependencies apache needs for it to run successfully on the web server. Without installation of these dependencies, apache will not start.

```
sudo yum update -y
sudo yum install httpd -y
sudo systemctl restart httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```

After Apache installation, I Confirmed that cgi.bin & html file were successfully created on the webserver after the Apache installation, run below from the Web server

`ls /var/www`

I was also able to confirm cgi.bin & html files exist on the NFS, run below on the NFS...

`ls /mnt/apps`

We can see that both `/var/www and /mnt/apps` contains same content. This shows that both mount points are connected via NFS. Seeing the same files – it means NFS is mounted correctly. I created a new file **touch test.txt** from one server and checked if the same file is accessible from other Web Servers.

![7_6](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/ac9e8e97-848b-4a12-83d9-2a2c6315e19d)

Next, I located the log folder for Apache on the Web Server and mounted it to NFS server’s export for logs. I aslo made sure the mount point will persist after reboot by updating the FSTAB.

Log folder for Apache is in this path /var/log/httpd. The httpd folder(log folder for Apache) should be empty, this was comfirm using `ls /var/log and sudo ls /var/log/httpd`

```
Script
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd
 
Executed
sudo mount -t nfs -o rw,nosuid 172.31.86.229:/mnt/logs /var/log/httpd
 
sudo ls /var/logs/httpd
```
 
I confirmed the mounting was successful by running `df -h` , I updated the fstab with below line

```
SCRIPT
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0

EXECUTED
sudo vi /etc/fstab
172.31.86.229:/mnt/logs /var/log/httpd nfs defaults 0 0
```

After confirmation the the Apache is active and running `sudo systemctl enable httpd` , i tried loading my web server public IP and got below page. This is a confirmation that our users can access the web server page, but more work is required to fine tune this default apache page.

![7_51](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/43dbb666-2339-40f9-9f18-bbda7097be14)


**I repeated steps 1-5 for the other 2 Web Servers**

**NOTE - BCOS the mount points on the webservers are connected via NFS, any activity from step 6 below will auto run on ALL the web servers**

### 6 - I forked the tooling source code 
from [Darey.io Github Account](https://github.com/darey-io/tooling) to my Github account. (Learn how to fork a repo [here](https://www.youtube.com/watch?v=f5grYMXbAV0))

I first ensured git is installed on my web server and also initialized, Then proceeded to run git clone. I also confirmed the download was successfull. Git is a tool used by software Engineers, it allows us to run source code management. Considering you a working in a team of software engineers. E.g, i can be working on login page mgt, another software Engineer is working on user access creation. There is need for source code mgt in this type of setting. How do we bring the 2 projects together to create a working software.

Below was done to install git

```
which git
sudo yum install git -y
git init
git clone https://github.com/darey-io/tooling.git
ls
which git
```

### 7. - I deployed the tooling website’s code to the Webserver. 

i ensured that the html folder from the tooling repository is deployed to **/var/www/html**

```
cd tooling
```

**Run below while on tooling directory** , the target is to copy ALL the content of html(inside the dowloaded repositoty - tooling) into the Apache folder(/var/www/html)...which gave us the default Apache page found in *step 5 , after i installed  Apache on the Web server*

`sudo cp -R html/. /var/www/html`

confirm the copying was successful, same content should be on below paths - run below while on tooling directory

```
ls /var/www/html    
ls html
```

- **Note 1**: 
I opened the TCP port 80 on the Web Server.

![7_7](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/00d921c9-9e22-4f18-8eb8-8b8678ff6253)

- **Note 2**: 
I tried launching my web server public IP on my browser, I still got same default Apache page populated, below was done to change the default Apache page

Below commands seeks to disabled SELinux `sudo setenforce 0`, then to make this change permanent, i opened following config file `sudo vi /etc/sysconfig/selinux` and then set SELINUX=disabled,  httpd(Apache) restart was done.

**Remember to exit the tooling directory**

```
sudo systemctl status httpd
cd ..
sudo setenforce 0
sudo vi /etc/sysconfig/selinux        => set SELINUX=enforcing to SELINUX=disabled
sudo systemctl start httpd
sudo systemctl status httpd
```
![7_777](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/72d75416-5d6c-409e-860c-6d41c1dac227)

By default, the apache page loads with `index.html` , this need to be changed to `index.php` - which is already downloaded on our tooling repo - see below

![7_71](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/c87111c7-5d7b-4585-a533-d3970ba61f73)

`sudo vi /etc/httpd/conf/httpd.conf` 

Run above, scroll down to the populate setting configurations, and locate below and make the change on the IF Module

![7_78](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/542c1970-9c2b-4727-9f23-479a7514e694)

 ... i proceeded to change from `index.html` , to `index.php`

 **NOTE** Considering my version of Linux: Red Hat Enterprise Linux 9, I had to use the tree command(see below) to locate `/etc/httpd/conf/httpd.conf` and make the required change of `index.html` , to `index.php`. You may need to do some search to locate the actuall folder where this change will happen. Below tree command came in handy in locating the folder where change was made....

```
sudo yum install tree
tree l6 ls /etc/httpd
```
 
After these changes, Apache was reloaded...
```
sudo systemctl reload httpd
sudo systemctl status httpd
```

On reloading the web server page(with web server public IP), below was displayed...

![7_79](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/fa48a2df-b869-46ce-beef-f766797aa55a) 

**NB** -  Always minimize the running a restart command in a production network - `sudo systemctl restart apache2` subcriber use of services are impacted, rather use reload .... `sudo systemctl reload apache2`

### 8 I Updated the website’s configuration to connect to the database

So basically, we seek to make the web server successfully connect the DB server which we have already setup, In the `/var/www/html` directory , edit the already written php script to connect to the database server, below CMD was used to achieve this

```
sudo vi /var/www/html/functions.php
```

![7_11](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/025ddda0-bbb1-41fc-be3f-3573c572916c)

After the modification , we can atttempt to connect to the database server from the web server using this CMD `mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql` 

But firstly, we need to install MYSQL on the web server in other to achieve this connection - see below.

### 11 Creating A table for the User on my Data Base
On the Database Server, We have created the Data Base, We have created the users, but we  have not created the table that will hold the data inputted to the DB. To achieve this...

I Applied tooling-db.sql script to my database using this command `mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

I first installed mysql on the web server which is running on Linux: Red Hat Enterprise Linux 9, then proceeded to apply the tooling script.

NOTE : This is script is for installation of MySQL version 8 on a RHEL 9 server. You can check youR Linux Version using  `hostnamectl`

```
sudo yum update
sudo yum install mysql-server -y   OR  sudo dnf install mysql-server -y
sudo systemctl start mysqld.service
sudo systemctl enable mysqld
sudo systemctl status mysqld
```

Below can also be used

```
sudo yum update
sudo dnf install mysql-server -y
sudo systemctl start mysqld.service
sudo systemctl enable mysqld
sudo systemctl status mysqld
```

Now that mysql has been installed on the web server, i will proceed to Apply tooling-db.sql script to my database

```
Script
mysql -h <databse-server-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql

Executed Script - ENSURE YOU ARE ON TOOLING DIRECTORY

cd tooling
mysql -h 172.31.91.150 -u webaccess -p tooling < tooling-db.sql
```
**Above script described**  => From my web server, I want to connect to mysql on the DB - 172.31.91.150, using user -u : webaccess , using password -p as my authentication, i want to connect directly to the tooling DB we have created on the DB Server, once connected, i want to run this sql file - tooling-db.sql, which has already been created and was part of the files downloaded on our tooling repo. 

**NOTE** , the tooling password captured above is not the real password, I got a prompt to input the correct password for the webaccess user already cretaed on the DB. Incaes the password prompt is not recieved when above command is ran on the Web server, ensure that MYSQL is running on the DB, and also the binding address has been set to 0.0.0.0

Also check the security inbound rules on the DB server, if ALL is OK, once you enter the correct password, you will see below marked green

![7_133](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/3fd65b92-e036-4ccc-bb13-eaf619a7061f)

Now I returned to my DATABASE SERVER and ran below commands

```
sudo mysql
show databases;
use tooling;       => connects you DIRECTLY to the tooling DB
show tables;       => This will now have users 
select * from users;  => This shows the table that has been created using sql file - tooling-db.sql while i was conected to the web sever
```
On mysql table on the DB Server, afer checking `select * from users;` the user displyed is same as the user diplayed when i run `sudo vi tooling-db.sql` from my web server.

### 10 Create in MySQL a new admin user with username : myuser and password: password

Now i proceeded to Simulate a sign up process by adding user credentials manually to the database, on the Data Base Server, below was executed...

`INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);`

![7_134](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/959c1f4c-7ce4-4e40-a7d5-00e1c3d1a2d5)


I made a log on attempt, and I was able to log in with the admin user from my tooling script above - user - admin, password - admin

# Congratulations! I was able to implemented a web solution for a DevOps team using LAMP stack with remote Database and NFS servers



# ASIDE INFO....
# Below is applicable when RHEL 8 is used

I tried to  the reload the Web Server test page which was displayed after Apache was install, and got same test page  display as the previous. While on my tooling directory, I located the test page using below ....

`ls /etc/httpd/conf.d/welcome.conf`

The content displayed when below is run is the same as what shows on the web server test page
`sudo vi /etc/httpd/conf.d/welcome.conf`

So I proceeded to rename the welcome page

`sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.backup`

Then i restarted the Apache

```
sudo systemctl restart httpd
sudo systemctl status httpd
```

Now attempt reloading the web server test page, if you get error - `Failed to connect to MySQL:Connection refused` , this means there is connectivity, but MySQL is refusing the connection. To troubleshoot this, 1st check if MYSQL is running,  `systemctl status mysql`    Next confirm if the require port numbers have been opened on your MYSQL server - try both  `telnet  localhost 3306`   & `telnet  localhost 33045`  and see the outputs, next is to check the inbound rule on the DB Server. I also ensured the binding address is set to 0.0.0.0

I reloaded the test web page again using the web server Public IP, and made log on attempt using the default username and password(admin/admin).

Now on the web server, i located the test web page currently displayed to my end users and renamed it


Below is the tooling website i seek to create, but i now have to install some PHP dependencies which will present the tooling wesite is the correct web page for my end users. These PHP dependencies are installed on the web server.

![7_14](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/40330c5f-5507-4d85-bf37-abd9432f2df5)

 ### 11 Install PHP dependencies on the Web Server

These PHP dependencies are installed on the web server.

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php -y
sudo yum module reset php -y
sudo yum module enable php:remi-7.4 -y
sudo yum install php php-opcache php-gd php-curl php-mysqlnd -y
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo systemctl status php-fpm
sudo setsebool -P httpd_execmem 1

Always ensure you restart Apache after making above changes, else the webpage will not load
sudo systemctl restart httpd
```


# CHECKING JENKINS workings2
# CHECKING JENKINS workings45
# check JENKINS PUSH TO NFS OVER SSH TEST TEST



                                                                               
