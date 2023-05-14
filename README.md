# Project-Base-Learning-7
DEVOPS TOOLING WEBSITE SOLUTION

## DEVOPS TOOLING WEBSITE SOLUTION

## PROJECT TASK
- Implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

## BACKGROUND KNOWLEDGE
- In previous [Project 6](https://www.darey.io/docs/project-6-step-1/) I implemented a WordPress based solution that was filled with content and can be used as a full fledged website or blog. Moving further I added some more value to my solutions that my DevOps team could utilize. I want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.

- The tools I want our team to be able to use are well known and widely used by multiple DevOps teams, so I will introduce a single DevOps Tooling Solution that will consist of:

- 1 [Jenkins](https://www.jenkins.io) - free and open source automation server used to build [CI/CD](https://en.wikipedia.org/wiki/CI/CD) pipelines.

- 2 [Kubernetes](https://kubernetes.io) – an open-source container-orchestration system for automating computer.

- 3 [Jfrog Artifactory](https://jfrog.com/artifactory/) – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.

- 4 [Rancher](https://www.rancher.com/products/rancher) – an open source software platform that enables organizations to run and manage [Docker](https://en.wikipedia.org/wiki/Docker_(software)) and Kubernetes in production.

- 5 [Grafana](https://grafana.com) – a multi-platform open source analytics and interactive visualization web application.

- 6 [Prometheus](https://prometheus.io) – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.

- 7 [Kibana](https://www.elastic.co/kibana/) – Kibana is a free and open user interface that lets you visualize your Elasticsearch [Elasticsearch](https://www.elastic.co/elasticsearch/) data and navigate the [Elastic](https://www.elastic.co/elastic-stack/) Stack.

Note: Do not feel overwhelmed by all the tools and technologies listed above, we will gradually get ourselves familiar with them in upcoming projects!

Side Self Study : Read about Network-attached storage (NAS), Storage Area Network (SAN) and related protocols like NFS, (s)FTP, SMB, iSCSI. Explore what Block-level storage is and how it is used by Cloud Service providers, know the difference from Object storage.
On the example of AWS services understand the difference between Block Storage, Object Storage and Network File System.

## Setup and technologies used in Project 7

- As a member of a DevOps team, I implemented a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

- In this project I implemented a solution that consists of following components:

- Infrastructure: AWS
- Webserver Linux: Red Hat Enterprise Linux 8
- Database Server: Ubuntu 20.04 + MySQL
- Storage Server: Red Hat Enterprise Linux 8 + NFS Server
- Programming Language: PHP
- Code Repository: [GitHub](https://github.com/darey-io/tooling)

- On the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using [Network File Sytem (NFS)](https://en.wikipedia.org/wiki/Network_File_System) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files.
- 
![7_1](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/0d10cd6d-5ecf-4cbc-a713-095719a85375)

It is important to know what storage solution is suitable for what use cases, for this – you need to answer following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Base on this you will be able to choose the right storage system for your solution.

### STEP 1 – ***PREPARE NFS SERVER***

1. - I Spinned up a new EC2 instance with RHEL Linux 8 Operating System.
For Rhel 8 server use this ami RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2 (ami-035c5dc086849b5de)

2. - Based on my LVM experience from **Project 6**, I Configured 3 LVM on the Server.

Install lvm2 package, create physical volume, create a volume group, create logical volume, format the disk with xfs file system 
```
sudo yum install lvm2
sudo lvmdiskscan
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
sudo pvs
```

```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
sudo vgs
```

```
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

3. - I ensured there are 3 **Logical Volumes**. ***lv-opt lv-apps, and lv-logs***

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

4. - I installed NFS server, configured it to start on reboot and made sure it is up and running

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
![7_2](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/19d68401-1eac-4032-9e91-23dcf13fb308)

5. -I exported the mounts for webservers’ ***subnet cidr*** to connect as clients. For simplicity, I installed all the three Web Servers inside the same subnet, but in production set up I would probably want to separate each tier inside its own subnet for higher level of security.

To check the ***subnet cidr*** – I opened my EC2 details in AWS web console and located ‘Networking’ tab and opened a Subnet link:

![7_3](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/bfa7b0f0-43b6-42e0-8194-59eaff2a3ac4)

- I make sure I set up permission that will allow the Web servers to read, write and execute files on NFS:

```
Change ownership of the directory
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

Allow read, write , Execute access on these directories
sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

Restart the NFS Server
sudo systemctl restart nfs-server.service
```

I Configured access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):

```
sudo vi /etc/exports

Script
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Script modified & Executed
/mnt/apps 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

Export modified file, so that the web servers will be able to access it
sudo exportfs -arv
```

6. - I checked which port is used by NFS and opened it using Security Groups (add new Inbound Rule)

- `rpcinfo -p | grep nfs` The port is **2049**

![7_4](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/d6b7520d-0a72-4c9b-927f-51608d55f0e9)

### STEP 2 — ***CONFIGURE THE DATABASE SERVER***

1. - I installed MySQL server on the Ubuntu OS

```
USED for installing MYSQL on a Ubuntu OS, Always run APT update 1st
sudo apt update
sudo apt install mysql-server -y
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

```
USED for installing MYSQL on a RedHart OS, Always run YUM update 1st
sudo yum update
sudo yum install mysql-server -y
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

2. - I created a database and named it **tooling**

3. - I created a database user and named it **webaccess**

4. - I granted permission to **webaccess** user on **tooling** database to do anything only from the webservers **subnet cidr**

```
sudo mysql
CREATE DATABASE tooling;
CREATE USER `webaccess`@`172.31.16.0/20` IDENTIFIED BY 'password';
GRANT ALL ON tooling.* TO 'webaccess'@'172.31.16.0/20';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit;
```
![7_5](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/2cd6341a-05be-46a6-8446-c7ab3bd3c1c9)


### Step 3 — ***PREPARE THE WEB SERVER***

- I ensured that the Web Servers can serve the same content from shared storage solutions, in this case – NFS Server and MySQL database.
Knowing that one DB can be accessed for **reads** and **writes** by multiple clients. For storing shared files that the Web Servers will use – I utilized NFS and mount previously created Logical Volume **lv-apps** to the folder where Apache stores files to be served to the users **(/var/www)**.

- This approach will make the Web Servers **stateless**, which means I will be able to add new ones or remove them whenever I need, and the integrity of the data (in the database and on NFS) will be preserved.

So basically, i seek to mount on my web server, all the logical volumes which i have created previously on the NFS server

- During the next steps I did the following:

 - Configured NFS client (this step must be done on all three servers)

 - Deployed a Tooling application to the Web Servers into a shared NFS folder

 - Configured the Web Servers to work with a single MySQL database

1. - I launched a new EC2 instance with RHEL 8 Operating System

2. - I installed NFS client using below command, note - without this installation, i will not be able to access the NFS Server from the web server

```
sudo yum install nfs-utils nfs4-acl-tools -y
```

3. - I mounted **/var/www/** and target the NFS server’s export for apps

```
sudo mkdir /var/www

Script
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

Executed Script
sudo mount -t nfs -o rw,nosuid 172.31.23.140:/mnt/apps /var/www
```

4. - I verified that NFS was mounted successfully by running `df -h`. I ensured that the changes will persist on Web Server after reboot:

- `sudo vi /etc/fstab`

- I added the following line inside the file in the fstab

```
Script
`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

Executed Script
172.31.23.140:/mnt/apps /var/www nfs defaults 0 0
```

5. - I installed  Apache and PHP, Remi's Repository  [Remi’s repository](http://www.servermom.org/how-to-enable-remi-repo-on-centos-7-6-and-5/2790/). Without the Apache you cant server content to your users, Nginx, Apache etc are the popular web servers clients out there.

```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y

sudo dnf module reset php -y

sudo dnf module enable php:remi-7.4 -y

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1
```

- **I repeated steps 1-5 for another 2 Web Servers**.

6. - I verified that Apache files and directories are available on the Web Server in **/var/www** and also on the NFS server in **/mnt/apps**. Seeing the same files – it means NFS is mounted correctly. I created a new file **touch test.txt** from one server and checked if the same file is accessible from other Web Servers.

![7_6](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/ac9e8e97-848b-4a12-83d9-2a2c6315e19d)

7. - I located the log folder for Apache on the Web Server and mounted it to NFS server’s export for logs. I repeated step **No 4** to make sure the mount point will persist after reboot. Log folder for Apache is in this path /var/log/httpd

```
ls /var/logs  - you will see httpd, this is the log folder for Apache, it shld be empty, check with below
sudo ls /var/logs/httpd

Script
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd
 
 Executed
 sudo mount -t nfs -o rw,nosuid 172.31.23.140:/mnt/logs /var/log/httpd
 
sudo ls /var/logs/httpd    confirm the mounting was successful
 
I updated the fstab with below line

sudo vi /etc/fstab
172.31.23.140:/mnt/logs /var/log/httpd nfs defaults 0 0
```

8. - I forked the tooling source code from [Darey.io Github Account](https://github.com/darey-io/tooling) to my Github account. (Learn how to fork a repo [here](https://www.youtube.com/watch?v=f5grYMXbAV0))

```
I first ensured git is installed on my web server and also initialized, Then proceeded to run git clone. I also confirmed the download was successfull
sudo yum install git -y
git init
git clone https://github.com/darey-io/tooling.git
ls
```


9. - I deployed the tooling website’s code to the Webserver. i ensured that the html folder from the repository is deployed to **/var/www/html**

```
cd tooling
ls /var/www    confirm there is a html folder here
sudo cp -R html/. /var/www/html   Run this while on tooling directory, the target is to copy ALL the content of html(inside the dowloaded repo - toolin


- **Note 1**: I opened the TCP port 80 on the Web Server.

- **Note 2**: If you encounter 403 Error – check permissions to your **/var/www/html** folder and also disable SELinux `sudo setenforce 0`




