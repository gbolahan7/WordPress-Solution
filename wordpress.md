## WordPress Solution

## Step 1 - Prepare a web server by launching an Instance on AWS. Next, create 3 EBS volumes of 10gb each and attach them to the Web server instance created

### Creae single partition on each of the 3 disks using 'gdisk' utility and follow the result prompts
`sudo gdisk /dev/xvdf`
`sudo gdisk /dev/xvdg`
`sudo gdisk /dev/xvdh`

### Use 'lsblk' utility to view the configured partition on the 3 disks

### Next, install Lvm2 and check for available partitions
`sudo yum install lvm2`

![lvm2](/images/lvm-install.PNG)

`sudo lvmdiskscan`

### Use pvcreate utility to mark each disks as physical volumes to be used by LVM(logical vol management).
`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

[lvm2](/images/phy-volume.PNG)

### Verify that the PV has been created successfully and use 'vgcreate' utility to add 3 PVs to a volume group named webdata-vg. A;so, verify the entire setup
`sudo pvs`
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![vol grp](/images/db-vol-grp.PNG)

`sudo vgs`

![vgs](/images/log%26vg.PNG)

`sudo lsblk`



### Format the logical volumes with ext4 filesystem
`sudo mkfs -t ext4 /dev/webdata-vg/app-lv`
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

### Create directories to store website files and store backup of log data respectively
`sudo mkdir -p /var/www/html`

`sudo mkdir -p /home/recovery/logs`

### Mount the /var/www/html on app-lv logical volume
`sudo mount /dev/webdata-vg/app-lv /var/www/html/`

### Back up all the files in log directory into home recovery logs as this is essential before mounting the file system
`sudo rsync -av /var/log/. /home/recovery/logs/`

### Mount the /var/log on logs-lv logical volume. (existing data will be deleted.hence the step above is vital)
`sudo mount /dev/webdata-vg/logs-lv /var/log`

### Update /etc/fstab file so the mount config persist after server restart. The UUID of the device will be used to update the /etc/fastab. N.B: remove the the leading and ending quotes [add ext4 defaults 0 0]
`sudo blkid`
`sudo vi /etc/fstab`

### Test the config and reload the daemon
`sudo mount -a`
`sudo systemctl daemon-reload`

## Step 2 - Prepare the Database Server

### Repeat same steps as the web server but mounting db-lv to /db directory instead
![db mount](/images/db-mount.PNG)


## Step 3 -  Install WordPress on Web Server EC2 

### Update repo and install wget,Apache and its dependencies. Then, start Apache.
`sudo yum -y update`

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

`sudo systemctl enable httpd`

`sudo systemctl start httpd`
![vgs](/images/httpd-run.PNG)

## Instal PHP and RHEL dependencies 

![php](/images/php-depenmdecies.PNG)

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo dnf update -y `

`sudo dnf upgrade --refresh -y `

`sudo dnf install epel-release `

`sudo subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms `

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm `

` $ sudo dnf install epel-release`

`sudo dnf update `
` sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm`
![vgs](/images/enable-remi.PNG)




`sudo yum module list php `
`sudo yum module reset php `
`sudo yum module enable php:remi-8.1 `
`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`
`sudo systemctl start php-fpm `
`sudo systemctl enable php-fpm`
`sudo setsebool -P httpd_execmem 1`

### Restart APACHE
`sudo systemctl restart httpd`

### Download wordpress and copy it to var/www/html
` mkdir wordpress`
`cd   wordpress`
`sudo wget http://wordpress.org/latest.tar.gz`
`sudo tar xzvf latest.tar.gz`
`sudo rm -rf latest.tar.gz`
`cp wordpress/wp-config-sample.php wordpress/wp-config.php`
`cp -R wordpress /var/www/html/`


## Configure SELinux Policies
`sudo chown -R apache:apache /var/www/html/wordpress`
`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`
`sudo setsebool -P httpd_can_network_connect=1`


## Step 4 - Install MySQL on DB Server EC2
`sudo yum update`
`sudo yum install mysql-server`

## Step 5 - Configure DB to work with WordPress
![vgs](/images/config-php.PNG)


`sudo mysql`
`CREATE DATABASE wordpress;`
`CREATE USER 'myuser'@'<Web-Server-Private-IP-Address>'IDENTIFIED BY 'mypass';`
`GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';`
`FLUSH PRIVILEGES;`
`SHOW DATABASES;`


## Step 6 - Configure Wordpress to connect to remote db
 ### Open port 3306 on DB server EC2 instance and allows access to the DB server only from web server IP addr.

 ### Next, Install MySQL client and test that it can be connected from web server to db server using mysql-client
 `sudo yum install mysql`
 `sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`

 ![vgs](/images/web-db-conn.PNG)

 ### Verify it can show database and then change permissions and config so Apache can use wordpress

 ### Finally, enable TCP port 80 in Web server instance from everywhere 0.0.0.0 and then try to access the browser via the link to the wordpress site
 `http://<Web-Server-Public-IP-Address>/wordpress/`

 ![wordpress](/images/wordpress.PNG)

 ![wordpress](/images/wp-web.PNG)

 ![wp](/images/wp-dashboard.PNG)

