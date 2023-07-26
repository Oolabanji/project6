## Implementing-Wordpress-Web-Solution
I create a EC2 instance server on AWS
On the EBS console, I created 3 storage volumes for the instance. This serves as additional external storage to the EC2 machine
![volumes](https://github.com/Oolabanji/test_/assets/136812420/b30d294d-27f9-4d2e-887c-25ea77b27348)
I attached the created volumes to the EC2 instance
![volumesattached](https://github.com/Oolabanji/test_/assets/136812420/6056c892-c066-4c52-8852-e9a570c2ec0d)
 I SSH into the instance and on the EC2 terminal, viewed the disks attached to the instance. This is achieved using the lsblk command.
 ![lsblk](https://github.com/Oolabanji/test_/assets/136812420/08883b11-9639-48bf-829a-add50ba0f0b4)
To see all mounts and free spaces on the server
![dfh](https://github.com/Oolabanji/test_/assets/136812420/5a3f95e7-736d-40e4-b8cb-114b2b535239)
I created single partitions on each volume on the server using gdisk 
![gdisk](https://github.com/Oolabanji/test_/assets/136812420/01fd4b6d-3777-4c57-bfa6-c7f9ab92c1b1)
![gdisk1](https://github.com/Oolabanji/test_/assets/136812420/6d16411d-f070-42bd-9376-09763fa12242)
Then I Installed LVM2 package for creating logical volumes on the linux server.
![lvm2](https://github.com/Oolabanji/test_/assets/136812420/0d47332d-f821-4ad9-9f90-460c4299809c)
Then I created Physical Volumes on the partitioned disk volumes using
'sudo pvcreate <partition_path>'
![pvcreate](https://github.com/Oolabanji/test_/assets/136812420/8678097c-a1cf-4307-b4b2-503522f8905a)
Next I added up each physical volumes into a volume group
'sudo vgcreate <grp_name> <pv_path1> ... <pv_path1000> '
![vgcreate](https://github.com/Oolabanji/test_/assets/136812420/11376b40-180a-48dc-a74f-524acf5bb3de)
I created Logical volumes for the volume group
'sudo lvcreate -n <lv_name> -L <lv_size> <vg_name>'
![lvcreate](https://github.com/Oolabanji/test_/assets/136812420/fdd50cf5-777f-4f0f-a1be-c749e2b60331)
The logical volumes are ready to be used as filesystems for storing application and log data.
I created filesystems on the both logical volumes
![mkfs](https://github.com/Oolabanji/test_/assets/136812420/ac7b6a71-b309-40d0-8b7f-b3a93a81b206)
The apache webserver uses the html folder in the var directory to store web content. I created this directory and also a directory for collecting log data of our application

![mkdirhtml](https://github.com/Oolabanji/test_/assets/136812420/9a5be2fc-3585-4c2a-89fa-6312193d10ee)
For our filesystem to be used by the server, I mounted it on the apache directory . Also I mounted the logs filesystem to the log directory
![rsync](https://github.com/Oolabanji/test_/assets/136812420/b03e3b79-5286-4de6-b94a-b49666cc8967)
I mounted logs logical volume to var logs and also restored back var logs data into var logs
![rsync2](https://github.com/Oolabanji/test_/assets/136812420/66ab1fce-5f68-4a38-99e2-a54619489de3)

### Persisting Mount Points
To ensure that all the mounts are not erased on restarting the server, I persist the mount points by configuring the /etc/fstab directory

'sudo blkid to get UUID of each mount points'
![blkid](https://github.com/Oolabanji/test_/assets/136812420/aaa8fa30-37b9-421a-a496-79343955aaed)
Then sudo vi /etc/fstab to edit the file
![fstab](https://github.com/Oolabanji/test_/assets/136812420/71cbc5c1-40e8-4d4c-a051-09226c63414a)
I tested mount point persistence
![sudomount](https://github.com/Oolabanji/test_/assets/136812420/472327b6-72ba-4f12-b420-86283658c2e9)
### Preparing DataBase Server
I repeated all the steps taken to configure the web server on the db server. And changed the apps-lv logical volume to db-lv


![databaseserver](https://github.com/Oolabanji/test_/assets/136812420/1d0d641e-d7d9-4624-9a5e-d2c8feea414c)
### Configuring Web Server
I run updates and installed httpd on web server
'yum install -y update'
'sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json'
Start web server
![httpd](https://github.com/Oolabanji/test_/assets/136812420/d5dbffa9-8ca3-4ba6-911d-450eef71dbdf)

Then I Installed php and its dependencies

'sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm'

'sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm'

'sudo yum module list php'

'sudo yum module reset php'

'sudo yum module enable php:remi-7.4'

'sudo yum install php php-opcache php-gd php-curl php-mysqlnd'

'sudo systemctl start php-fpm'

'sudo systemctl enable php-fpm'

'setsebool -P httpd_execmem 1'

Restarting Apache: 'sudo systemctl restart httpd'

I downloaded wordpress and moved it into the web content directory

'mkdir wordpress'

'cd   wordpress'

'sudo wget http://wordpress.org/latest.tar.gz'

'sudo tar xzvf latest.tar.gz'

'sudo rm -rf latest.tar.gz'

'cp wordpress/wp-config-sample.php wordpress/wp-config.php'

'cp -R wordpress /var/www/html/'

Configure SELinux Policies

'sudo chown -R apache:apache /var/www/html/'

'sudo chcon -t httpd_sys_rw_content_t /var/www/html/'

'sudo setsebool -P httpd_can_network_connect=1'

### Installing MySQL on DB Server

'sudo yum update'

'sudo yum install mysql-server'

To ensure that database server starts automatically on reboot or system startup

'sudo systemctl restart mysqld'

'sudo systemctl enable mysqld'

I started database server

![mysql](https://github.com/Oolabanji/test_/assets/136812420/a488502b-5bce-4775-9256-d62cf540662a)

### Setting Up DB Server
![SETTINGDB](https://github.com/Oolabanji/test_/assets/136812420/41febad5-e288-49fe-86a1-e7a4a394e225)

i ensured that I add port 3306 on the db server to allow the web server to access the database server.
![set3306](https://github.com/Oolabanji/test_/assets/136812420/53244709-ce2b-4872-823c-f7c4c187eb16)

#### Connecting Web Server to DB Server
I installed mySQl client on the web server so I can connect to the db server

'sudo yum install mysql'

'sudo mysql -u admin -p -h 172.31.13.234'
![connecting to dbserver](https://github.com/Oolabanji/test_/assets/136812420/a77e77db-f293-4d94-aad9-791f1c347d83)

On the web browser, access web server using the public ip address of the server
![wordpress](https://github.com/Oolabanji/test_/assets/136812420/19debb60-5663-4c8d-b08e-269baf7bbf7c)

![wordpresswelcome](https://github.com/Oolabanji/test_/assets/136812420/5576b139-b431-42d4-92a0-610aa918e282)





