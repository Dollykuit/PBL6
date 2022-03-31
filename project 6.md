
# Project 6 - WEB SOLUTION WITH WORDPRESS

## Step 1 - Preparing the web server 
* Created a Red Hat AWS EC2 instance that will serve as a Web server
* Created 3 volumes in the same AZ as the Web Server EC2, each of 10 GiB.
* Attach all three volumes one by one to your Web Server EC2 instance

<img width="960" alt="Screenshot 2022-03-31 003346" src="https://user-images.githubusercontent.com/98477745/160947830-de0f9eaf-b081-49b9-9989-d6dce644c19c.png">
* connected to the Web server and ran the below cmdlet to confirm the volumes are attached

    `sudo gdisk /dev/xvdf`
     
    
  * used `df -h` to check the free space in the server and also the mount points available
  
  <img width="960" alt="Screenshot 2022-03-31 004710" src="https://user-images.githubusercontent.com/98477745/160948892-4a3649d7-457f-4049-81d8-82d6e1aced7b.png">
  
  * Use gdisk utility to create a single partition on each of the 3 volume
  `sudo gdisk /dev/xvdf`
  
  <img width="960" alt="Screenshot 2022-03-31 005144" src="https://user-images.githubusercontent.com/98477745/160949303-cfb1be99-eb5e-49a0-ab4c-bea48f4baf1b.png">
  
  `sudo gdisk /dev/xvdg`
  
<img width="960" alt="Screenshot 2022-03-31 005519" src="https://user-images.githubusercontent.com/98477745/160949549-c196e29f-5af2-47b6-bea9-d6ba1cc74e9a.png">

`sudo gdisk /dev/xvdh`

![image](https://user-images.githubusercontent.com/98477745/160949868-b483d3f0-69d3-46a2-a6f9-87321d30cb66.png)

* ran `lsblk` to confirm the partitions have been successfully created

<img width="960" alt="Screenshot 2022-03-31 005853" src="https://user-images.githubusercontent.com/98477745/160950273-80cc63ca-ee21-4110-a476-efaef5d42373.png">

* install LVM2 to create logical volumes
`sudo yum install lvm2`

<img width="960" alt="Screenshot 2022-03-31 010625" src="https://user-images.githubusercontent.com/98477745/160950574-6904c786-505c-46ff-8e34-0d40b2986daf.png">

* ran `sudo lvmdiskscan` to confirm the available partitions


<img width="960" alt="Screenshot 2022-03-31 010517" src="https://user-images.githubusercontent.com/98477745/160950728-0b9d5b86-afc6-4ce3-a7a1-fccf1df1d61d.png">

* we then created physical volumes on the partitions using the below:

`sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1`


<img width="960" alt="Screenshot 2022-03-31 011450" src="https://user-images.githubusercontent.com/98477745/160951225-c914a003-a0d4-4d09-8d7c-132e81659dc6.png">

* We then confirmed the Physical volumes have been created using `sudo pvs`


<img width="960" alt="Screenshot 2022-03-31 011708" src="https://user-images.githubusercontent.com/98477745/160951443-b6736e7e-5368-48f4-8762-100fb8008222.png">

* We then Used `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG ''webdata-vg''

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

<img width="960" alt="Screenshot 2022-03-31 011708" src="https://user-images.githubusercontent.com/98477745/160953091-a9779671-3bf2-4995-a8f0-f12ff957f082.png">

Verify that your VG has been created successfully by running 

`sudo vgs`

<img width="960" alt="Screenshot 2022-03-31 013637" src="https://user-images.githubusercontent.com/98477745/160953237-b8d8490e-e2c0-4959-ba20-f0d82236c718.png">


* Use `lvcreate` utility to create 2 logical volumes apps-lv and log-lv. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

`sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg`

<img width="960" alt="Screenshot 2022-03-31 014329" src="https://user-images.githubusercontent.com/98477745/160953793-91faafc1-cbd0-49df-91bf-5493970b37ef.png">

Verify that your Logical Volume has been created successfully by running 

`sudo lvs`

<img width="959" alt="Screenshot 2022-03-31 014524" src="https://user-images.githubusercontent.com/98477745/160953973-b6b753ec-2aa7-49ec-9238-62042d789d01.png">

* Run the below command tpo verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk `


<img width="960" alt="Screenshot 2022-03-31 015019" src="https://user-images.githubusercontent.com/98477745/160954378-3ff19ea7-4467-4f4c-a576-545b98be7e90.png">

* Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`


<img width="960" alt="Screenshot 2022-03-31 015446" src="https://user-images.githubusercontent.com/98477745/160954801-259d3991-4ad2-46f8-a689-a383828f82a2.png">

* Create /var/www/html directory to store website files

`sudo mkdir -p /var/www/html`

* Create /home/recovery/logs to store backup of log data

`sudo mkdir -p /home/recovery/logs`

* Mount /var/www/html on apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

* Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`
<img width="958" alt="Screenshot 2022-03-31 015959" src="https://user-images.githubusercontent.com/98477745/160955256-621b01a1-9402-433f-891f-ad922fe960a0.png">

* Update /etc/fstab  using your own UUID and rememeber to remove the leading and ending quotes.
* The UUID of the 2 logical volume of the apps-lv and the logs-lv can be found using `sudo blkid`

<img width="960" alt="Screenshot 2022-03-31 021142" src="https://user-images.githubusercontent.com/98477745/160956462-37a6a777-861f-4b5f-8d75-a320fccb4cf2.png">

added the below UUID to the /etc/fstab file

UUID=5b40a6f8-6348-44a2-a613-6180960f8654
UUID=60e08037-74dc-4b79-b931-ee316d11cbda

`sudo vi /etc/fstab`


<img width="959" alt="Screenshot 2022-03-31 022224" src="https://user-images.githubusercontent.com/98477745/160957417-dcb8fbec-1b2a-4b26-ad4a-f2536a139cf3.png">

* Tested the configuration and reloaded the daemon.

 ` sudo mount -a
 sudo systemctl daemon-reload`
 
 Verified setup is running `df -h`
 
 
<img width="960" alt="Screenshot 2022-03-31 022836" src="https://user-images.githubusercontent.com/98477745/160957932-72919856-94ca-4295-8695-8c91009ba2e0.png">

## Step 2 - Preparing the Database Server

Created a second Red hat EC2 instance that will have the role Database Server
Created 3 EBS volume in the same AZ as the Database server 
 * Attached each of the EBS volume to the DB server
 * Connected to the Daabase Server and ran the below command to confirm the volumes are attached and are showing.
 
 `lsblk`

<img width="959" alt="Screenshot 2022-03-31 023940" src="https://user-images.githubusercontent.com/98477745/160958974-45b876c1-c96a-42a4-99a6-27c2280c079f.png">
 
 * We use `df -h` to check the free space in the server and also to mount points available.

* We then use gdisk command to create a partition to each volume

`sudo gdisk /dev/xvdf`
<img width="960" alt="Screenshot 2022-03-31 024743" src="https://user-images.githubusercontent.com/98477745/160959852-d6307bd2-a2f4-4db4-b1a6-792f79eff2e1.png">

`sudo gdisk /dev/xvdg`

<img width="960" alt="Screenshot 2022-03-31 025153" src="https://user-images.githubusercontent.com/98477745/160960299-458d6af9-af8a-4223-91ca-e70bcd12729c.png">

`sudo gdisk /dev/xvdh`

<img width="960" alt="Screenshot 2022-03-31 025350" src="https://user-images.githubusercontent.com/98477745/160960513-2b574d5a-13ac-4455-b6d8-e8aa419486e8.png">

* Ran the `lsblk` to confirm again


<img width="960" alt="Screenshot 2022-03-31 025528" src="https://user-images.githubusercontent.com/98477745/160960689-ff485c7d-a068-4142-999e-6325396f1b3a.png">

* Installed the LVM2 to create logical Volumes

`sudo yum install lvm2`

<img width="960" alt="Screenshot 2022-03-31 025841" src="https://user-images.githubusercontent.com/98477745/160961117-00d8d853-5d9c-4fc2-a3e3-4be77f4c67ad.png">

* Ran `sudo lvmdiskscan` to confirm the available partitions


<img width="960" alt="Screenshot 2022-03-31 030210" src="https://user-images.githubusercontent.com/98477745/160961381-5d00f500-c0d0-4ca5-8992-209fc021bd0c.png">

* Created PVs on the partitions using the below:

`sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1`

* we confirmed the PVs have been created using `sudo pvs`
* Added all the PVs to a VG, we did that using the below and the name should be "webdata-vg"

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

<img width="959" alt="Screenshot 2022-03-31 031138" src="https://user-images.githubusercontent.com/98477745/160962311-677504da-270e-4f81-a3db-9319fee9586b.png">

* We use `sudo vgs` to confirm the volume group has been created
* Use the "lvcreate" utility to create 2 LV 

`sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg`

* Verify that the Logical Volume has been created successfully by running

`sudo lvs`


<img width="960" alt="Screenshot 2022-03-31 032412" src="https://user-images.githubusercontent.com/98477745/160963615-a2ee471b-c42a-4e91-9842-6df90ad52661.png">

* We used the mkfs.ext4 to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`



<img width="958" alt="Screenshot 2022-03-31 033056" src="https://user-images.githubusercontent.com/98477745/160964271-b9321bf5-6b19-4c71-a4d3-45d16ebf530e.png">

* Created ext/db directory ext to store db files 

`sudo mkdir /db`

* Mounted /db on apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /home/ec2-user/db`

* Updated the /etc/fstab file so that the mount configuration will persist after restart of the server
* Found the UUID of the logical volume of the db-lv using

`sudo blkid`

edited the following UUID into the /etc/fstab file

UUID=34c300d7-24a6-46ba-b50e-8248a1e513f6
UUID=a932628b-159f-47f0-9d84-63e104157bc2

`sudo vi /etc/fstab`

* Tested the configuration and reloaded the deamon

`sudo mount -a`
`sudo syatemctl daemon-reload`

* Verified the setup by running `df -h`


## Step 3 - Install Wordpress on your EC2 Web Server

* Updated the repository
`sudo yum -y update`

* Installed wget, Apache and it’s dependencies
`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

* Started Apache

`sudo systemctl enable httpd
sudo systemctl start httpd`

* installed PHP and it’s depemdencies


`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1`

* Restarted Apache

`sudo systemctl restart httpd`

* Downloaded wordpress

`mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz`

<img width="960" alt="Screenshot 2022-03-31 035927" src="https://user-images.githubusercontent.com/98477745/160967507-d32a064e-9f14-4dca-a0d2-e19b18f2e378.png">

* Extracted the compressed word press installation file and then removed the compressed file

`sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz`
  
* Changed the name of the config file to "wp-config.php" and then copied the file to /var/www/html/ path


`cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/`
  
* Configured SELinux Policies

` sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1`
  
## Step 4 - Install MySQL on your BD Server EC2

`sudo yum update
sudo yum install mysql-server`

<img width="960" alt="Screenshot 2022-03-31 041307" src="https://user-images.githubusercontent.com/98477745/160969210-fe518dbb-e9ae-4b84-bf57-39f34a605383.png">

  


<img width="960" alt="Screenshot 2022-03-31 041434" src="https://user-images.githubusercontent.com/98477745/160969236-8ad2cee8-7984-400c-b812-e83f582699fe.png">

* Verify the server is up and running by using sudo systemctl status mysqld

`sudo systemctl restart mysqld
sudo systemctl enable mysqld`


<img width="960" alt="Screenshot 2022-03-31 042058" src="https://user-images.githubusercontent.com/98477745/160969935-660afd3e-579a-4b61-aeb1-ab5aba109f58.png">

## Step 5 - Configure DataBase to work with wordpress

`sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit`

<img width="960" alt="Screenshot 2022-03-31 042454" src="https://user-images.githubusercontent.com/98477745/160970233-bf1d17e8-9343-4e10-a1d5-a3f15581f108.png">
   
## Step 6 - Configure WordPress to connect to remote database.
    
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32
    
Created an inbound rule allowing connection only from web server
 <img width="960" alt="Screenshot 2022-03-31 043156" src="https://user-images.githubusercontent.com/98477745/160970981-398abfe3-59f7-445c-af62-fe952a11f331.png">
    
* Installed MySQL client and tested that it can connect from the Web Server to the DB server by using mysql-client

`sudo yum install mysql`
    
 <img width="960" alt="Screenshot 2022-03-31 043513" src="https://user-images.githubusercontent.com/98477745/160971592-f03a443f-294a-402c-af3c-9de74a88109d.png">
    
 `sudo mysql -u myuser -p -h <DB-Server-Private-IP-address>`
    
 <img width="960" alt="Screenshot 2022-03-31 043623" src="https://user-images.githubusercontent.com/98477745/160971753-ab677455-d73a-4690-b3cd-336c35b2721e.png">
    
 * Verify if you can successfully execute `SHOW DATABASES;` command and see a list of existing databases.
    
 * Changed permissions and configuration so Apache could use WordPress
    
 <img width="959" alt="Screenshot 2022-03-31 044453" src="https://user-images.githubusercontent.com/98477745/160972304-1b864384-ef57-43b8-bba9-66bc45ab7c2c.png">
    
 * From the webserver EC2, edited the wp-config.php file to pupoluatre the DB information using the below
 changed directory into wordpress and ran `sudo vi wp-config.php`
    
 Edited the below information
    
 `/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'myuser' );

/** Database password */
define( 'DB_PASSWORD', 'password' );

/** Database hostname */
define( 'DB_HOST', '172.31.91.232' );`
    
Note this was the user that was created (myuser) in the Database that was granted permission to the database that was created.
Then saved and accessed from the browser the link to my wordpress
http://<Web-Server-Public-IP-Address>/wordpress/
<img width="960" alt="Screenshot 2022-03-30 062659" src="https://user-images.githubusercontent.com/98477745/160974563-df5cd012-dd6b-4585-be07-c422e9862cad.png">

 

    
 

    
 
<img width="960" alt="Screenshot 2022-03-30 063000" src="https://user-images.githubusercontent.com/98477745/160974600-536273b6-aef0-4fb2-8c39-ff2f91c9828d.png">

<img width="960" alt="Screenshot 2022-03-31 050849" src="https://user-images.githubusercontent.com/98477745/160974706-5d44044a-0abc-4cbb-8f66-6415dc48441c.png">

# THE END.
   




