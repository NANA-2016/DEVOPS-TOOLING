# DEVOPS-TOOLING

# NFS SERVER CONFIGURATION.

Opening nfs server on RHEL 8 on AMI on AWS cloud.

![Screenshot 2024-01-09 163616](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/9323b397-83fe-4d6b-a243-dfca203cc8f2)


Next an lvm  is installed usind the command   <sudo yum install lvm2>

and configured where partitions prior to its installation then later physical volumes are creates

and put in one volume group which is then divided into logical volumes with equal storage space.

Commands used are as shown below.

sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

sudo lvcreate -n apps-lv -L 9G webdata-vg
sudo lvcreate -n logs-lv -L 9G webdata-vg
sudo lvcreate -n opt-lv -L 9G webdata-vg


Aphoto of the coplet set up is shown below wherre the results are ftrom <sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk >

![Screenshot 2024-01-09 170746](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/b9d1dbc0-8224-4101-9f3c-5e23a4aa7303)

![Screenshot 2024-01-09 170900](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/8b378a73-5e56-42b7-9ed2-9046cd25ca2b)

The files are then configured to xfs format using the commands below.

sudo mkfs -t xfs /dev/webdata-vg/apps-lv
sudo mkfs -t xfs /dev/webdata-vg/logs-lv
sudo mkfs -t xfs /dev/webdata-vg/opt-lv

mnt folders are then ccreate to form mounting points for apps,logs and opt using sudo mkdir command.

 The command for mounting all the tree files are as follows.

 sudo mount /dev/webdata-vg/apps-lv /mnt/apps
sudo mount /dev/webdata-vg/logs-lv /mnt/logs
sudo mount /dev/webdata-vg/opt-lv /mnt/opt

![Screenshot 2024-01-09 171735](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/8aea6a31-cc20-4f4e-81ab-2b6aa869f151)

NFSserver is installed and left active to run.

sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
 
![Screenshot 2024-01-09 172133](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/632e18e8-7e15-4694-97e8-dbceb658f6a4)

Change of ownership and change of premissions is done so as the webservers can edit files.

![Screenshot 2024-01-09 172518](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/afdf2d1e-4b43-4d4a-8325-e39b139d00bc)

further configurations are made on the /etc/expo file to allow smooth renning of all the servers and the database as well.

 The screen shot below show the changes made and the commands run as well as giving us the port that NFS server is 
 
 accessible from which is port 2049

 ![Screenshot 2024-01-09 173257](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/d40b45c7-2e09-43de-8580-84ebc646c45c)


sudo vi /etc/exports

/mnt/apps 172.31.32.0/20
(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.32.0/20
 (rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.32.0/20
 (rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv

rpcinfo -p | grep nfs

Configurations are made on the inbound rules to allow smooth running of all the servers using the subnet CIDR ports tcp 111,2049 and upd 111,2049

![Screenshot 2024-01-09 173647](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/bc595b9c-2257-421a-bab2-e18e53149670)

#DATABASE SERVER

 Data base server is on UBUNTU on AWSclound.

 For any confugurations to be done MYSQL SERVER  first installed soo as to install the database and run the following  commands
 
 create database;
  create user'webaccess'@'172.31.32.0/20' identified by 'password';
   grant all privileges on tooling.* to 'webaccess'@'172.31.32.0/20';
   FLUSH PRIVILEGES

To seee if these has been sucessfull,
 the commands below are run and the screen shot shows the expected results.

mysql> show databases;
mysql> show tables;
mysql> select * from users;
 
** the  command for creating user is a s follows**

create user'<any username>'@'<sub net cidr of the nsf server>' identified by 'password';

![Screenshot 2024-01-09 180315](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/7f428e10-6176-4f2a-b0c8-423df4b7aa9c)


 # CONFIGURATION OF THE NFS CLIENTS

  Here we are going to confugure three clients with same commands.

  They are  all set up in AMI RHEL 8 just as the nsf srver is .

  ![Screenshot 2024-01-09 184642](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/19bf13c1-3372-4be4-a50b-e6cc9db6553d)

## Install nfs client 

   sudo yum install nfs-utils nfs4-acl-tools -y is a command used to install NSF client

   ![image](https://github.com/NANA-2016/DEVOPS-TOOLING/assets/141503408/65a81d28-3d72-4425-b087-4af05322509d)

sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

Afolder is creater where the /mnt/apps is mounted which carries all apps .

df -h shows id wthe mounting was successful

sudo vi /etc/fstab
later a configuratin on the file etc/fstab is made using the NSF server private ip adderess.
 
sudo vi /etc/fstab ( command )
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0 ( configuration on the etc/fstab file which is directly cpoied in to the file on vi)


## REMI REPOSITORY COMMANDS 
All this command have too be run one by one to ensure all the required coomands have been inacted and httpd is active ar running. as shown in the screen shot after the last coomand.
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1

 All ythe above are run in all the three servers.

  To ensure everything is working check if the apache files in WEBSERVER ARE ALSO IN ALL THE THTREE SERVERS.

  IN server 1 text.txt
  server2 text.txt 2
  server 3 text.txt 3 are expected to be seen on he NFS swerver as well as the webservers.

   ALL WEB SERVERS ARE ACCESSIBLE THROUGH PORT 80

   Forking the tooling website fron the website  is the next step  where git is installed and git clonning done as shown below on all webservers [https://github.com/darey-io/tooling.git]
   
       the next step is to deploy  tooling website code in to var/www/html foldrer

    For the webserver to run smothel we need to disable selinux and set it to disabled on setenforce 0
     using the coomand /etc/sysconfig/selinux and restart httpd

      install my sql client ansd use the database server  to be able to access the database serve in tooling database. created earlier.
sudo yum install mysql -y

 cd tooling
 mysql -h 172.31.34.136 -u webaccess -p tooling <tooling-db.sql

 Now you can access the website using the public ip adresses of the three webservers.


    


  

  



