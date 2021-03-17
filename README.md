# MOODLE 3.9+ Ubuntu 20
The perfect Moodle, by Rene Solis

# SETUP
```
UBUNTU amigable para Unoconv con permisos

sudo apt install apache2 mysql-client mysql-server php libapache2-mod-php
dpkg-reconfigure dash

service apparmor stop
update-rc.d -f apparmor remove 
apt-get remove apparmor apparmor-utils

apt-get -y install ntp
apt install apache2 mysql-client mysql-server php libapache2-mod-php

mysql_secure_installation
*/*/*/
Enter current password for root (enter for none): <-- press enter
Set root password? [Y/n] <-- y
New password: <-- Enter the new MariaDB root password here
Re-enter new password: <-- Repeat the password
Remove anonymous users? [Y/n] <-- y
Disallow root login remotely? [Y/n] <-- y
Reload privilege tables now? [Y/n] <-- y
*/*/*/

sudo apt install graphviz aspell ghostscript clamav php7.4-pspell php7.4-curl php7.4-gd php7.4-intl php7.4-mysql php7.4-xml php7.4-xmlrpc php7.4-ldap php7.4-zip php7.4-soap php7.4-mbstring
sudo service apache2 restart


sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
/// Scroll down to the [mysqld] section and under Basic Settings add the following line under the last statement.///// SIN LINEAS EN BLANCO
default_storage_engine = innodb
innodb_file_per_table = 1
#innodb_file_format = Barracuda (MARCA ERROR)
sudo service mysql restart


sudo mysql -u root -p
CREATE DATABASE a----- DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
use a-----;
CREATE USER 'a-----'@'%' IDENTIFIED BY '------';
GRANT ALL PRIVILEGES ON a-----.* TO 'a-----'@'%' WITH GRANT OPTION;
flush privileges;
quit

sudo apt install git
cd /opt
git clone git://git.moodle.org/moodle.git
cd moodle
sudo git branch -a
sudo git branch --track MOODLE_39_STABLE origin/MOODLE_39_STABLE
sudo git checkout MOODLE_39_STABLE
sudo cp -R /opt/moodle /var/www/html/
sudo mkdir /var/www/moodledata21
sudo chown -R www-data /var/www/moodledata21
sudo chmod -R 777 /var/www/moodledata21
sudo chmod -R 777 /var/www/html/moodle

***sudo chmod -R 0755 /var/www/html/moodle  **CERRAR

crontab -u www-data -e

 * * * * * /usr/bin/php  /var/www/html/moodle/admin/cli/adhoc_task.php --execute --keep-alive=59 >/dev/null


apt install unoconv
mkdir  /var/www/html/.config
chown www-data:www-data   /var/www/html/.config
sudo chmod -R 777 /var/www/html/.config
sudo chown www-data /var/www
nano /etc/php/7.4/apache2/php.ini 
https://docs.moodle.org/35/en/Universal_Office_Converter_(unoconv)#Run_a_unoconv_listener


    Type "sudo nano /etc/php5/apache2/php.ini"
    Press Ctrl and W and type "post_max_size"
    Change the value to the number of Mb you want your site to accept as uploads
    Press Ctrl and W and type "upload_max_filesize"
    Change the value to the number of Mb you want your site to accept as uploads
    Press Ctrl and W and type "max_execution_time"
    Change the value to 600
    Press Ctrl and O
    Press Ctrl and X
    Type sudo apachectl restart

////////////////////////////////////////////////
CACHE
////////////////////////////////////
apt install php7.4-dev
sudo apt-get update

cd /tmp
git clone https://github.com/krakjoe/apcu

Now build the php7.4-apcu extension and install it.
cd apcu
phpize
./configure
make
sudo make install

sudo service apache2 reload

FINALMENTE
Making use of APC within Moodle

The first thing you will need to do is create an APC cache store instance. This is done through the Cache configuration interface.

    Log in as an administrator and go to 'Caching > Configuration' in the Site administration.
    Locate the APC row within the Installed cache stores table. You should see an "Add instance" link within that row. If not then the APC extension has not being installed correctly.
    Click "Add instance".
    Give the new instance a name and click "Save changes". You should be directed back to the configuration page.
    Locate the Configured cache store instances table and ensure there is now a row for you APC instance and that it has a green tick in the ready column.

Once done you have an APC instance that is ready to be used. The next step is to map definitions to make use of the APC instance.

    Locate the known cache definitions table. This table lists the caches being used within Moodle at the moment. For each cache you should be able to Edit mappings.
    Find a cache that you would like to map to the APC instance and click Edit mappings.
    One the next screen proceed to select your APC instance as the primary cache and save changes.
    Back in the known cache definitions table you should now see your APC instance listed under the store mappings for the cache you had selected. You can proceed to map as many or as few cache definitions to the APC instance as you see fit.

That is it! you are now using APC within Moodle. 

////////////////////////////////////
REDIS https://linuxize.com/post/how-to-install-memcached-on-ubuntu-20-04/
sudo apt update
sudo apt install redis-server
sudo systemctl enable redis-server
sudo apt install php-redis

sudo vim /etc/redis/redis.conf
   maxmemory 256mb
   maxmemory-policy allkeys-lru

*EN CASO DE UPGRADE A PHP USAR: (sudo phpenmod -v 7.4 -s ALL redis)
redis-cli info
redis-cli info stats
redis-cli info server
CREAR LA LLAVE Y PONERSELA EN LA CONFIGURACION

IR A MOODLE Y AGREGAR UNA NUEVA INSTANTANCIA, PONER 127.0.0.1
///////////////////////////////////////
sudo apt update
sudo apt install memcached libmemcached-tools
(sudo nano /etc/memcached.conf)
(sudo ufw allow from 192.168.100.30 to any port 11211)
sudo apt install php-memcached
systemctl reload apache2
IR y activarlo
http://localhost/moodle/cache/admin.php
```



# GoogleChrome for Linux and the MAX SIZE for backups upper from 300Mb
```bash
sudo apt -f install
sudo dpkg --configure -a
sudo apt -f install
sudo apt-get install gdebi-core -y 
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb 
sudo gdebi google-chrome-stable_current_amd64.deb 
sudo gedit /usr/share/applications/google-chrome.desktop  
ECHO
ECHO /// MAX SIZE para respaldos arriba de 300Mb ///
ECHO https://docs.moodle.org/310/en/File_upload_size
ECHO
nano /etc/php/7.4/apache2/php.ini 
ECHO MODIFICAR:"post_max_size" 1Gb,  "upload_max_filesize" 1GB, "max_execution_time" a 600
service apache2 restart 
ECHO /// restart Mysql5 ///
nano  /etc/mysql/mysql.conf.d/mysqld.cnf 
ECHO MODIFICAR: max_allowed_packet      = 100M
service mysql restart
```

# DIRECTORIES
```bash
ECHO MOODLEDATA:
chmod -R 0770 moodledata/
chown -R nobody:apache moodledata
ECHO Instalar Pluggins:
chmod -R 0777 moodle
ECHO exit
chmod -R 0755 /var/www/html/moodle
```

# CRON Cli
```
CRON CLI
/usr/bin/php  /var/www/html/moodle/admin/cli/cron.php
```

# ERRORS from MoodleData Cache Purge
Si purgamos el cache, no correr el script, mejor DIRECTAMENTE borrar "cache" moddledata, marca errores aveces
