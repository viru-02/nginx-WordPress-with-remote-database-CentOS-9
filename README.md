# nginx-WordPress-with-remote-database-CentOS-9
Wordpress Install On Nginx Server And Use Also RD, Os - CentOS - 9 
Nginx WordPress with remote database CentOS 9
CentOS Stream 9 me bhi dnf package manager use hota hai.

sudo dnf update -y
sudo dnf install epel-release -y
sudo dnf install nginx php php-fpm php-mysqlnd php-gd php-xml php-mbstring php-json -y
Agr Output me killed aaye mtlb memory km hai to swap memory bna lo
Swap file add karo
free -h
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show
free -h
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab (Permanent swap (reboot ke baad bhi active rahe)

Ab firse try karo install
sudo dnf install nginx php php-fpm php-mysqlnd php-gd php-xml php-mbstring php-json -y

Create Remote Database (AWS RDS)
AWS Console me jao → Search “RDS” → open RDS Dashboard → Create database

Database Engine Select karo

Engine type: MySQL
Version: Latest (for example: MySQL 8.0.x)
Templates: “Free tier” select karo (agar free trial account hai).

Database Settings

DB instance identifier: wordpress-db
Master username: admin
Master password: koi strong password (note kar lena)
Confirm password: same password

Instance Configuration
DB instance class: db.t3.micro (Free tier)
Storage: Default (20GB okay hai)
Enable “Storage autoscaling” (optional).

Connectivity Settings

Virtual Private Cloud (VPC): default
Public access: ✅ Yes (Important — taaki EC2 se connect ho sake)
VPC security group:
Create new security group (e.g. wordpress-db-sg)

Security Group Configuration

RDS ke security group me ek inbound rule add karna zaruri hai:

AWS Console → EC2 → Security Groups
Apna wordpress-db-sg select karo
Inbound rules → Edit inbound rules

Add rule:
Type: MySQL/Aurora
Port: 3306
Source: EC2 instance ka public IP address ya EC2 ka security group

Database Create karo (inside RDS)

MySQL client install karo

sudo dnf install mysql -y (sql Client Install)
mysql -h <your-rds-endpoint> -u admin -p (end point on your db)

Login hone ke baad ye command chalao →
CREATE DATABASE wordpress_db;
EXIT; (this cmd make a empty db)

WordPress ko RDS se connect karo

cd /var/www/wordpress
sudo vim wp-config.php (in this file save db info)

define('DB_NAME', 'wordpress_db');
define('DB_USER', 'admin');
define('DB_PASSWORD', 'your_rds_password');
define('DB_HOST', 'your-rds-endpoint.rds.amazonaws.com');
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');

And Then Check & Setup Wordpress

http://<your-ec2-public-ip>/

Verify PHP can reach RDS

echo "<?php
\$link = mysqli_connect('wordpress-db.c9k4aksigmnp.us-west-1.rds.amazonaws.com', 'admin', 'q:UY]6\-8Y6^', 'wordpress_db');
if (!\$link) { die('❌ Connection error: ' . mysqli_connect_error()); }
echo '✅ Connected successfully to RDS!';
?>" | sudo tee /var/www/wordpress/dbtest.php

Now Open Browser And Check 
http://<your-ec2-public-ip>/dbtest.php

Make sure WordPress is using the correct wp-config.php

Sometimes Nginx serves files from another directory, not from /var/www/wordpress.

grep root /etc/nginx/conf.d/*.conf
















