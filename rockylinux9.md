```bash
https://ciq.com/blog/how-to-install-the-phpmyadmin-web-based-mysql-or-mariadb-gui-on-rocky-linux/
https://www.cyberciti.biz/faq/disable-selinux-on-centos-7-rhel-7-fedora-linux/
https://docs.moodle.org/403/en/Git_for_Administrators
https://webhostinggeeks.com/howto/how-to-install-lamp-on-centos-7/



#Disabling SELinux permanently
# Edit the /etc/selinux/config file, run:
sudo vi /etc/selinux/config
#Set SELINUX to disabled:
SELINUX=disabled
# Update system
sudo su
dnf update -y


sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

#Command to install the Remi repository configuration package:
    dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm -y

#If no version is installed, command to install the php stream default profile:
    dnf module install php:remi-8.3 -y

#Command to install additional packages (xxx for SAPI or extension name):
#    dnf install php-xxx

# MariaDB REPO WIZARD a actualizar, MOODLE requiere 10.06 o superior.
# https://mariadb.org/download/?t=repo-config&d=CentOS+Stream&v=10.11&r_m=xtom_fre

nano /etc/yum.repos.d/MariaDB.repo
---- Generado con el Wizard ------
# MariaDB 10.11 CentOS repository list - created 2024-02-04 01:46 UTC
# https://mariadb.org/download/
[mariadb]
name = MariaDB
# rpm.mariadb.org is a dynamic mirror if your preferred mirror goes offline. See https://mariadb.org/mirrorbits/ for details.
# baseurl = https://rpm.mariadb.org/10.11/centos/$releasever/$basearch
baseurl = https://mirrors.xtom.com/mariadb/yum/10.11/centos/$releasever/$basearch
# gpgkey = https://rpm.mariadb.org/RPM-GPG-KEY-MariaDB
gpgkey = https://mirrors.xtom.com/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck = 1
------------------------------------




sudo yum install git MariaDB MariaDB-client mariadb-server httpd php php-mysql php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap -y

sudo systemctl enable --now mariadb

dnf install php-zip
dnf install php-sodium
dnf install php-gd
dnf install php-iconv
dnf install php-mbstring
dnf install php-curl
dnf install php-openssl
dnf install php-tokenizer
dnf install php-soap
dnf install php-ctype
dnf install php-zlib
dnf install php-simplexml
dnf install php-redis -y
dnf install php-spl

dnf install php-pcre
dnf install php-dom
dnf install php-xml
dnf install php-xmlreader

dnf install php-intl -y

dnf install php-json
dnf install php-hash
dnf install php-fileinfo
dnf install php-exif
dnf install php-memory_limit
dnf install php-file_upload
dnf install php-file_uploads
dnf install php-opcache.enable
dnf install php-opcache

php --modules
php --version


sudo systemctl start httpd.service
sudo systemctl enable httpd.service
sudo systemctl start mariadb
sudo systemctl enable mariadb.service


sudo mariadb-secure-installation 

nano /etc/php.ini
# The behaviour of these functions is affected by settings in php.ini.
# https://bizanosa.com/moodle-server-requirements/
#
date.timezone = "UTC"  
#memory_limit=256M
#file_uploads=ON
#zend_extension=/full/path/to/opcache.so

#[opcache]
opcache.enable = 1
opcache.memory_consumption = 128
opcache.max_accelerated_files = 10000
opcache.revalidate_freq = 60

; Required for Moodle
opcache.use_cwd = 1
opcache.validate_timestamps = 1
opcache.save_comments = 1
opcache.enable_file_override = 0
///////////////////////////////


# Creacion de la BD
mysql -u root -p
CREATE DATABASE ________ DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    create user '_______'@'localhost' IDENTIFIED BY '_____';
    GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON ________.* TO '______'@'localhost';
  FLUSH PRIVILEGES;
  exit
------

cd
dnf install unoconv -y


# MOODLE 4.x

sudo mkdir /var/www/moodledocs24a
sudo chown -R apache:apache  /var/www/moodledocs24a
sudo chmod 777  /var/www/moodledocs24a

-----

# Instalacion de Moodle 4.x via Git
cd /var/www
git clone git://git.moodle.org/moodle.git
cd moodle
git branch -a
git branch --track MOODLE_403_STABLE origin/MOODLE_403_STABLE 
git checkout MOODLE_403_STABLE         

```