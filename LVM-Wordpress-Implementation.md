# logical volume management in linux

<b>Logical Volume Management (LVM)</b> is a technology used in Linux to manage disk storage more flexibly and efficiently. It abstracts the physical storage devices (hard drives or SSDs) into logical volumes, providing a higher level of control and flexibility for managing storage.

![logical volume management in linux](images/LVM.jpg)

<h3>Key components of LVM:</h3>

<b>Physical Volume (PV):</b> A physical storage device like a hard drive or SSD.

<b>Volume Group (VG):</b> A collection of physical volumes grouped together. It acts as a pool of storage.

<b>Logical Volume (LV):</b> Slices of volume groups that function like partitions. They can be dynamically resized and moved.

<b>LVM Thin Provisioning:</b> This feature allows you to create thinly provisioned logical volumes. It efficiently utilizes available storage space and allows overcommitment of disk space.

<h3>Advantages of LVM:</h3>

<b>Flexibility:</b> LVM allows you to resize, move, and manage storage without the need to repartition disks.

<b>Snapshot</b>: You can create point-in-time snapshots of logical volumes for backup or testing purposes.

<b>Volume Striping:</b> Data can be striped across multiple physical volumes for improved performance.

<b>Redundancy:</b> LVM supports mirroring (RAID1) and striping with parity (RAID5) for improved data protection.

<b>Dynamic Resizing</b>: You can resize logical volumes while the system is running, making it easier to manage storage demands.

<b>Ease of Management:</b> LVM provides a command-line interface and utilities to manage storage operations.

<h2>LVM Implementation</h2>

In this hands-on demonstration of logical volume management, we are going to allocate resources for both a web server and a database server within separate logical volume infrastructures. The web server will host a WordPress website, establishing communication with the MySQL server located on the database server. Our approach involves utilizing the accessible EC2 instances along with the Elastic block storage (volume) assets supplied by AWS.

<h3>Database Server Set up</h3>

We start by launching an AWS EC2 instance (Redhat) for the database server, set the inbound rule to allow traffic from SSH on port 22, TCP port 3306.

![logical volume management in linux](images/db-1.jpg)

Before leaving the AWS console, create three(3) volumes of 10G disk size each, and attach them to the already running web server instance.

![logical volume management in linux](images/db-2.jpg)

Now, sign into the webserver through SSH 

![logical volume management in linux](images/db-3.jpg)

Chcek the attached volume witht the command below:

    sudo lsblk

![logical volume management in linux](images/db-4.jpg) 

Now the three disks (xvdf, xvdg, and xvdh) have been successfully attached to the machine.
We'll use the <mark>gdisk</mark> command to create partion for all the disks.

We start by taking one disk at a time.

    sudo gdisk /dev/xvdf

![logical volume management in linux](images/db-5.jpg) 

Type <mark>n</mark> to create new partition

Type <mark>1</mark> to create just one partition

Leave the next two prompts as default options by click <mark>Enter</mark> to bypass.

Set the logical file system by typing <mark>8e00</mark> and press Enter

Input <mark>p</mark> to preview settings

After previewing, input <mark>w</mark> to write all changes (save)

comfirm all by selecting yes(y)

![logical volume management in linux](images/db-6.jpg)


<b>Now we do for disk <mark>xvdg</mark></b>

    sudo gdisk /dev/xvdg

<b>Now we do for disk <mark>xvdh</mark></b>

    sudo gdisk /dev/xvdh

After create partitions for all disks, to confirm this partition configuration settings type the command below

    sudo lsblk

![logical volume management in linux](images/db-7.jpg)

The next step is to install <mark>lvm2</mark></b>. It is na utility used in Linux systems to create and manage logical volumes. It provides a layer of abstraction between the physical storage devices (hard drives, SSDs) and the filesystems, allowing for more flexible and efficient management of storage resources. Here's what LVM2 does when setting up an LVM infrastructure

    sudo yum install lvm2

![logical volume management in linux](images/db-8.jpg)

The verify that lvm2 has been successfully installed, use this command the path that the package reside:

    which lvm

![logical volume management in linux](images/db-9.jpg)

From the partitioned disks, we will create physical volumes with <mark>pvcreate</mark></b>. Instead of repeating same three times, we do it once with a command.

    sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

![logical volume management in linux](images/db-10.jpg)

Run the <mark>pvs</mark></b> command to view available physical volumes and their properties.

    sudo pvs

![logical volume management in linux](images/db-11.jpg)

Now, we will create a volume group consisting of the three pysical volumes we created earlier. With the <mark>vgcreate</mark></b> command we will get this done, and assign a name <b>vg-database</b> to it.

    sudo vgcreate vg-database /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

![logical volume management in linux](images/db-12.jpg)

Run the <mark>vgs</mark></b> command to view available volume groups and their properties.

    sudo vgs 

![logical volume management in linux](images/db-13.jpg)

With <marklvcreate</mark> we will create ond logical volume from the <b>vg-database</b> . One from for website data and the other for backup purposes.

    sudo lvcreate -n database-lv -L 20G vg-database

![logical volume management in linux](images/db-14.jpg)

Run the <mark>lgs</mark></b> command to view available logical volume and their properties.

    sudo lvs

![logical volume management in linux](images/db-15.jpg)

Follow by using <mark>mkfs.ext4</mark> to format the logical volumes to ext4 filesystem

    sudo mkfs.ext4 /dev/vg-webdata/database-lv

![logical volume management in linux](images/db-16.jpg)

Now, will we create a directory for our database in the root directory.

    sudo mkdir /db
    
Next, we will mount the /dev/vg-database/database-lv on the /db

To verify, whether the webdata-lv has been mounted, do this:

    sudo df -h

    sudo mount /dev/vg-database/database-lv /db

Copy and paste the command below to get the UUID for both app-lv and log-lv

    sudo blkid

 ![logical volume management in linux](images/db-17.jpg)

    Sudo vi /etc/fstab

 ![logical volume management in linux](images/db-18.jpg)

 To check whether the UUIDs have been successfully propagated, copy and paste this:

![logical volume management in linux](images/db-19.jpg)

    sudo mount -a 

Now reload daemon for proper functioning

    sudo systemctl reload daemon

Now our infrastruction is ready. The next step is to install upgrade the machine and install mysql server.To update, do this:

    sudo yum update -y

 ![logical volume management in linux](images/db-20.jpg)

And install mysql server with this:

    sudo yum install mysql-server

![logical volume management in linux](images/db-20.jpg)

Now, we configure the mysql server application with the line below

    sudo mysql_secure_installation

For the purpose of this exercise, we not use a secure security password, we'll allow the default setting by inputting any other letter aside 'y'

![MYSQL DATABASE ARCHITECTUTRE](images/no.jpg)

After the installation and configuration settings of MYSQL-serevr, we then navigate into the mysql console by typing the command below

     mysql -u root -p

Since we are in the mysql console, we will create a database 'employees_data'

    mysql> CREATE DATABASE wordpress;

Create a user

    mysql> CREATE USER 'wordpress'@'%' IDENTIFIED BY 'password';

Grant the user all privilrges

    mysql> GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'%' WITH ALL GRANT OPTION;
Next, flush all privileges

     mysql> FLUSH PRIVILEGES;

![MYSQL DATABASE ARCHITECTUTRE](images/sudo-mysql.jpg)

Lastly on the database server, we modify mysql configuration to allow traffic from outside localhost.

    sudo vi /etc/my.cnf.d

copy and paste this and then close the editor. 


Now we are done with the database server, and it is ready to store data.


<h3>Web Server Set up</h3>

We start by launching an AWS EC2 instance (Redhat) for the web server, set the inbound rule to allow traffic from SSH on port 22, HTTP on port 443.

![logical volume management in linux](images/wb1.jpg)

Before leaving the AWS console, create three(3) volumes of 10G disk size each, and attach them to the already running web server instance.

![logical volume management in linux](images/wb2.jpg)

Now, sign into the webserver through SSH 

    ssh -i "Myserver-key.pem" ec2-user@ec2-35-173-254-11.compute-1.amazonaws.com

Chcek the attached volume witht the command below:

    sudo lsblk

![logical volume management in linux](images/wb3.jpg) 

Now the three disks (xvdf, xvdg, and xvdh) have been successfully attached to the machine.
We'll use the <mark>gdisk</mark> command to create partion for all the disks.

We start by taking one disk at a time.

    sudo gdisk /dev/xvdf

Type <mark>n</mark> to create new partition 

Type <mark>1</mark> to create just one partition
 

Leave the next two prompts as default options by click <mark>Enter</mark> to bypass.

Set the logical file system by typing <mark>8e00</mark> and press Enter

Input <mark>p</mark> to preview settings 

After previewing, input <mark>w</mark> to write all changes (save) 

comfirm all by selecting yes(y)

![logical volume management in linux](images/wb4.jpg)


<b>Now we do for disk <mark>xvdg</mark></b>

    sudo gdisk /dev/xvdg

![logical volume management in linux](images/wb5.jpg)

<b>Now we do for disk <mark>xvdh</mark></b>

    sudo gdisk /dev/xvdh

![logical volume management in linux](images/wb6.jpg)

After create partitions for all disks, to confirm this partition configuration settings type the command below

    sudo lsblk

![logical volume management in linux](images/wb7.jpg)

The next step is to install <mark>lvm2</mark></b>. It is na utility used in Linux systems to create and manage logical volumes. It provides a layer of abstraction between the physical storage devices (hard drives, SSDs) and the filesystems, allowing for more flexible and efficient management of storage resources. Here's what LVM2 does when setting up an LVM infrastructure

    sudo yum install lvm2 -y

![logical volume management in linux](images/wb8.jpg)

The verify that lvm2 has been successfully installed, use this command the path that the package reside:

    which lvm

From the partitioned disks, we will create physical volumes with <mark>pvcreate</mark></b>. Instead of repeating same three times, we do it once with a command.

    sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

Run the <mark>pvs</mark></b> command to view available physical volumes and their properties.

    sudo pvs

![logical volume management in linux](images/wb9.jpg)

Now, we will create a volume group consisting of the three pysical volumes we created earlier. With the <mark>vgcreate</mark></b> command we will get this done, and assign a name <b>vg-webdata</b> to it.

    sudo vgcreate vg-webdata /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

Run the <mark>vgs</mark></b> command to view available volume groups and their properties.

    sudo vgs

![logical volume management in linux](images/wb10.jpg)

With <marklvcreate</mark> we will create two logical volume from the <b>vg-webdata</b> . One from for website data and the other for backup purposes.

    sudo lvcreate -n webdata-lv -L 14G vg-webdata

![logical volume management in linux](images/wb11.jpg)

    sudo lvcreate -n logs-lv -L 14G vg-webdata

![logical volume management in linux](images/wb12.jpg)

Run the <mark>lgs</mark></b> command to view available logical volume and their properties.

    sudo lvs

![logical volume management in linux](images/wb13.jpg)

Follow by using <mark>mkfs.ext4</mark> to format the both logical volumes to ext4 filesystem

    sudo mkfs.ext4 /dev/vg-webdata/webdata-lv

And

    sudo mkfs.ext4 /dev/vg-webdata/logs-lv

![logical volume management in linux](images/wb14.jpg)

Now, will we create a directory where our website will be located.

    sudo mkdir -p /var/www/html

Follow by creating temporary backup directory for log contents

    sudo mkdir -p /home/recovery/logs
    
Next, we will mount the /dev/vg-webdata/webdata-lv on the /var/www/html but before then, we have to ascertain that the /var/www/html directory is empty.

    sudo ls -l /var/www/html

![logical volume management in linux](images/wb15.jpg)

Now, we can do the mounting since the directory we are omunting on is empty

    sudo mount /dev/vg-webdata/webdata-lv /var/www/html

![logical volume management in linux](images/wb16.jpg)

To verify, whether the webdata-lv has been mounted, do this:

    sudo df -h

![logical volume management in linux](images/wb17.jpg)

For the logs-lv, we are going to mount it on /var/log directory, but the directory contains some vital content. Before mounting, we have to backup the content in the /home/recovery/logs directory we create, mount it and then restore the content.
To backup the content, do this;

    sudo rsync -av /var/log /home/recovery/logs

Then, we mount:

    sudo mount /dev/vg-webdata/logs-lv /var/log

Let's return the content into /var/log directory

    sudo rsync -av /home/recovery/logs/ /var/log

 ![logical volume management in linux](images/wb18.jpg)

To cobfirm if log-lv has been successfully mounted 

    sudo df -h

Copy abd paste the command below to get the UUID for both app-lv and log-lv

    sudo blkid

 ![logical volume management in linux](images/wb19.jpg)

    Sudo vi /etc/fstab

 ![logical volume management in linux](images/wb20.jpg)

 To check whether the UUIDs have been successfully propagated, copy and paste this:

    sudo mount -a 

Now reload daemon for proper functioning

    sudo systemctl reload daemon

Now, we are done with the webserver infrastructure.

    sudo yum update -y

 ![logical volume management in linux](images/wb21.jpg)

Nex, we will install the following packages; php and it dependencies, wget, httpd, mysql-client, and unzip

    sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json unzip mysql-server

Using the wget and unzip utilities, we will download wordpress and unzip it in the /var/www/html directory. use the below command to download wordpress

    sudo wget https://wordpress.org/latest.zip

Unzip the latest.zip file with unzip utility

    sudo unzip latest.zip

![logical volume management in linux](images/wb22.jpg)

Create wp-config.php from wp-config-sample.php

    sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php 

Modify the wp-config.php with the database user created on the database server

    sudo vi wordpress/wp-config.php

![logical volume management in linux](images/wb23.jpg)

Save and exit the editor's terminal. Copy all content in the wordpress folder to /var/www/html directory

    sudo cp worpress/. -r /var/www/html

Do this to comfirm the transfer

    ls -l -r /var/www/html

![logical volume management in linux](images/wb24.jpg)

Code run the commands below

    sudo chown -R apache:apache /var/www/html
    sudo chcon -t httpd_sys_rw_content_t /var/www/html -R
    sudo setsebool -P httpd_can_network_connect_db=1
    sudo setsebool -P httpd_can_network_connect=1
    sudo systemctl enable httpd
    sudo systemctl start httpd
    sudo systemctl enable mysqld
    sudo systemctl start mysqld

Then, enter the web server url in a browser, and to should see the a web page similar to the image below.

![logical volume management in linux](images/wb25.jpg)

# Conclusion

If you are able to get the webpage similar to image above, you have successfully set up logical volume management systems to host a database server and a webserver. Congratulations to you!