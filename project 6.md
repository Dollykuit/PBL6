
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







