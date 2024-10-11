### DEVOPS TOOLING WEBSITE SOLUTION
In my previous project I implemented a WordPress-based solution that is now ready to be used as a fully functional website or blog. In this project, I will be adding more value to this solution by implementing a tooling website solution that makes access to DevOps tools within the corporate infrastructure easily accessible.

In this project, I will implement a solution that consist of the following components:
1. Infrastructure (AWS)
2. Webserver Linux: RedHat Enterprise Linux 8
3. Database Server: Ubuntu 20.04 + MySQL
4. Storage Server: Redhat Enterprise Linux 8 + NFS Sever
5. Programming Language: PHP
6. GitHub Code 

# Prerequisites
1.  Knowledge of AWS core services and CLI
2.  Basic knowledge of Linux commands and how to manage storage on a Linux server.
3.  Basic knowledge of Network-attached storage (NAS), Storage Area Networks (SAN), and related protocols like NFS, FPT, SFTP, SMB, iSCSI.
4.  Knowledge of Block-level storage and how it is used on the Cloud.

# Architecture
I will be implementing a solution that comprises of multiple web servers sharing a common database and also accessing the same files using Network File System (NFS) as shared file storage.
![reference image](/Pictures/pic44.PNG)
It is important to know what storage solution is suitable,for what use cases, for this - you need to answer following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Base on this you will be able to choose the right storage system for your solution.

## Step By Step Procedure
# 1. Partitioning and Mounting
1. Spin up a new EC2 instance with RHEL Linux 8 Operating System and name it as NFS (Network File System) then create and attach 3 volumes to it ![reference image](/Pictures/pic27.PNG). Make sure they're in the same zone for you to be able to attach them.
2. Coonect to your terminal and prepare the NFS Server,start by partitioning your disks ![reference image](/Pictures/pic2.PNG), then install *lvm2* ![reference image](/Pictures/pic4.PNG), create *pvs* ![reference image](/Pictures/pic5.PNG), create *vgs* ![reference image](/Pictures/pic7.PNG), create *lvs* [reference image](/Pictures/pic8.PNG) then  configure LVM in the server by formatting the disk as xfs.![reference image](/Pictures/pic9.PNG)
3. Mount the logical volume and verify it was successfully mounted ![reference image](/Pictures/pic10.PNG)
4. Run *sudo blkid* to get the UUID of the mount part, open and paste the UUID in the fstab file using *sudo vi /etc/fstab* ![reference image](/Pictures/pic11.PNG) 
5. Relaod daemon using *sudo mount -a*  and *sudo systemctl daemon-reload*


# 2. Install NFS server, configure it to start on reboot and make sure it is u and running 
1. Run update using *sudo yum -y update*
2. Then install the NFS Server using *sudo yum install nfs-utils -y*
3. finally start, enable and check the status using *sudo systemctl start nfs-server.service*, *
sudo systemctl enable nfs-server.service* and *sudo systemctl status nfs-server.service* ![reference image](/Pictures/pic12.PNG)  ![reference image](/Pictures/pic13.PNG)
**NOTE**:  If the output of the systemctl status nfs-server.service command shows that the NFS service is "active (exited)", it means that the service is running, but it is not currently doing anything. This is normal behavior for the NFS service when there are no active NFS client connections.
5. Export the mounts for webservers *subnet cidr* to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security. To check your *subnet cidr* - open your EC2 details in AWS web console and locate 'Networking' tab and open a Subnet link and you should see this ![reference image](/Pictures/pic14.PNG)
6. Setup that will allow the web server to read, write and execute files on NFS and restart the NFS server using ![reference image](/Pictures/pic15.PNG)
7.   Configure access to NFS for clients within the same subnet using the *subnet Cidr*, run *sudo vi /etc/exports* then paste the follwing code but using your own *Cidr* ![reference image](/Pictures/pic16.PNG) and export ![reference image](/Pictures/pic17.PNG)
8.   Check which port is used by NFS and open it using Security Groups. In order for NFS server to be accessible from your client, you must also open the following ports: TCP 111, UDP 111 in addition to the NFS port. use this to check for NFS port *rpcinfo -p | grep nfs* ![reference image](/Pictures/pic18.PNG), ![reference image](/Pictures/pic19.PNG)

# 3. Configure the Database Server
1. Launch a new EC2 instance with Ubuntu or RedHat but I used Ubuntu though name it DB Server, connect to your terminal and install MySQL server ![reference image](/Pictures/pic20.PNG)
2. login into the mysql console and Create a database and name it *tooling* ![reference image](/Pictures/pic21.PNG)
3. Create a database user called *webaccess*
4. Grant permission to this *webaccess* user to have full permissions on the “tooling” database only from the subnet cidr. After you done you should be able to do this ![reference image](/Pictures/pic22.PNG)
# 4. Prepare the Web Servers

In this step, we will be launching three web servers. We need to make sure that the web servers can serve the same content from shared storage solutions, which in this case are the MySQL database and NFS server.

For storing shared files that our Web Servers will use, we will utilize NFS and mount previously created logical Volume 8lv-apps8 to the folder where Apache stores files to be served to the users (*/var/www*).

This approach will make our Web Servers *stateless*, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.
1. Launch a new EC2 instance with RHEL 8 Operating System Update and Install NFS client ![reference image](/Pictures/pic23.PNG)
2. Mount */var/www/* and target the NFS server’s export for apps using *sudo mkdir /var/www* and *sudo mount -t nfs -o rw,nosuid <NFS Server Private IP>:/mnt/apps /var/www* and Make sure that the changes will persist on the Web Server after reboot by adding the below text to the /etc/fstab file and reload daemon. confirm your changes by running  *df -h*![reference image](/Pictures/pic25.PNG) and ![reference image](/Pictures/pic24.PNG)
3.  Install Remi’s repository, Apache and PHP By ruunig the following command:
1. sudo yum install git -y
2.  install httpd -y
3.  sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y
4.  sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
5.  sudo dnf module reset php -y -y
6.  sudo dnf module enable php:remi-7.4 -y
7.  sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y -y
8. sudo systemctl start php-fpm
9.  sudo systemctl enable php-fpm
10. sudo setsebool -P httpd_execmem 1 ![reference image](/Pictures/pic29.PNG)
# REPEAT THE ABOVE STEPS FOR THE OTHER 2 WEB SERVERS
4. Verify that Apache files and directories are available on the Web Server in */var/www* and also on the NFS server in */mnt/apps*. If you see the same files, it means NFS is mounted correctly. You can test this by creating a new file from one web server and check if it is accessible from other web servers. ![reference image](/Pictures/pic30.PNG), ![reference image](/Pictures/pic31.PNG), ![reference image](/Pictures/pic32.PNG)
5. Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step 2 to make sure the mount point will persist after reboot.run *sudo mount -t nfs -o rw,nosuid <NFS server Private IP>:/mnt/logs /var/log/httpd* ![reference image](/Pictures/pic33.PNG)
# 5. Deploy a Tooling Application to our Web Server into a Shared NFS Folder
1. Fork the tooling source code from <https://github.com/StegTechHub/tooling>to your Github account. Learn how to fork here<https://www.youtube.com/watch?v=f5grYMXbAV0>
2.  Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to */var/www/html* ![reference image](/Pictures/pic34.PNG)
3.  Open TCP port 80 on the Web Server. ![reference image](/Pictures/pic35.PNG)
4.  attempt to restart httpd service, it very likely that it will fail to start at this point stating that httpd service is unable to write to the log directory. If you encounter this error, check permissions to your */var/www/html* folder to ensure that it is own by root. 
Disable SELinux by running *sudo setenforce 0*
To make this change permanent, open following config file *sudo vi /etc/sysconfig/selinux* and set *SELINUX=disabledthen* restart httpd. ![reference image](/Pictures/pic37.PNG) 
5. Restart httpd service by running *sudo systemctl restart httpd* ![reference image](/Pictures/pic37.PNG)
6.  Update the website’s configuration file (*/var/www/html/functions.php*) to connect to the database use your Darabase Private IP Address here ![reference image](/Pictures/pic38.PNG)
7.  Apply tooling-db.sql script to your database using this command *mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql*
8.  Create in MySQL a new admin user with username: myuser and password: password, but before then *cd tooling* then install MySQL client using *sudo yum install mysql* connect to mySQL server from the webserver using *webaccess* user created earlier and the Privaye IP of the DB server ![reference image](/Pictures/pic41.PNG) ![reference image](/Pictures/PIC39.PNG)
**NOTE** make sure you change your bind address in the *mysqld.conf* to your DB Private IP Address so you can connect remotely the restart mysql service ![reference image](/Pictures/pic40.PNG)
9. run this code in the mysql console  *INSERT INTO 'users' ('id', 'username', 'password', 'email', 'user_type', 'status') VALUES -> (1, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');*
10. Open the website in your browser *http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php* and make sure you can login into the websute with myuser user. ![reference image](/Pictures/pic42.PNG) ![reference image](/Pictures/pic43.PNG)

# We have successfully implemented and deployed a DevOps tooling website solution that makes access to DevOps tools within the corporate infrastructure easily accessible. This comprises multiple web servers sharing a common database and also accessing the same files using Network File System (NFS) as shared file storage

**Credit**
<https://steghub.com/lessons>

   
