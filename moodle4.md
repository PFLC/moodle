# MOODLE  4 Almalinux 9.2
# Parcialmente basado en https://www.howtoforge.com/how-to-install-moodle-elearning-platform-on-rocky-linux-8/
# Se corrige el php8.1 a 8.0 otro tutorial

# Actualizar la instalacion de 9Gb USB Bootable
```bash
dnf update -y
sudo dnf install wget curl nano unzip yum-utils git -y
echo "Checar el Firewall e instalar herramienta GUI de Firewall poner EXTERNO como default HTTP/HTTPS"
sudo firewall-cmd --state
sudo firewall-cmd --permanent --list-services

echo "Instalacion de REDIS / Memcache"
sudo systemctl start redis
sudo systemctl enable redis
```
TODO
https://www.howtoforge.com/redis-made-easy-a-step-by-step-guide-to-installing-on-almalinux-9/

# Install and Configure PHP verson 8.0 (ver moodle.org soporte php) AlmaLinux trae 8.1 no compatible
#  Installing the EPEL and Remi Repository, For this, we use the PHP 7.4 and 8.1 packaged by Remi,
# First, let us install the EPEL repository, UPSCALE 9.0 then select 8.0
```bash

dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
dnf module list php
dnf module install php:remi-8.0
dnf update
php -v

echo "The default version is 8.1. Enable Remi's PHP 8.0 repository." 
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.0

```
# Instalar complementos 350Mb de php8 pero falla ASPELL

```bash
sudo dnf install graphviz  ghostscript clamav php-fpm php-iconv php-curl php-mysqlnd php-cli php-mbstring php-xmlrpc php-soap php-zip php-gd php-xml php-intl php-json php-sodium php-opcache

echo "Install ASPELL" via https://almalinux.pkgs.org/9/almalinux-crb-x86_64/aspell-0.60.8-8.el9.x86_64.rpm.html
dnf --enablerepo=crb install aspell

```

# Modificar PHP.ini 256M para subir respaldos
```bash
sudo nano /etc/php.ini
    upload_max_filesize = 256M
    post_max_size = 256M
    max_input_vars = 5000

sudo nano /etc/php-fpm.d/www.conf
    ; Unix user/group of processes
    ; RPM: apache user chosen to provide access to the same directories as httpd
     user = nginx
     ; RPM: Keep a group allowed to write in log dir.
     group = ngix
     ...
     listen.owner = nginx
     listen.group = nginx
     listen.mode = 0660
     //Comentar//
      ;listen.acl_users = apache,nginx
      
echo "PHP permisos y servicios"
chown -R nginx:nginx /var/lib/php/session/
sudo systemctl enable php-fpm --now

```
# Instalar MYSQL
```bash
sudo dnf install mysql-server
mysql --version
sudo systemctl enable mysqld --now
sudo mysql_secure_installation

     Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: (Type 2) PONER TIPO 2...
     ....
     Poner contrasena dificil y darle como 5 "y"

sudo mysql -p
 mysql > CREATE DATABASE moodledb DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
 mysql >create user 'moodleuser'@'localhost' IDENTIFIED BY '......passwrod....';
 mysql >GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodledb.* TO 'moodleuser'@'localhost';
 mysql > FLUSH PRIVILEGES;
 mysql >exit
```

# Instalar ngix web server
```bash
sudo nano /etc/yum.repos.d/nginx.repo
/////////
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
///////// Salvar y salir/////

sudo dnf install nginx
nginx -v

sudo mkdir /var/www/html/moodle
sudo chown -R $USER:$USER /var/www/html/moodle
cd /var/www/html/moodle
git clone https://github.com/moodle/moodle.git .
git branch -a
git branch --track MOODLE_402_STABLE origin/MOODLE_402_STABLE
git checkout MOODLE_402_STABLE
sudo mkdir /var/moodledata
sudo chown -R nginx /var/moodledata
sudo chmod -R 775 /var/moodledata
sudo chmod -R 755 /var/www/html/moodle
echo "Configurar"
cd /var/www/html/moodle
cp config-dist.php config.php

```

# Certificado
```bash
echo "Debe estar apagado todo webserver"
sudo certbot certonly --standalone --agree-tos --no-eff-email --staple-ocsp --preferred-challenges http -m moodle@aula.lazarocardenas.edu.mx -d aula.lazarocardenas.edu.mx

echo "The above command will download a certificate to the /etc/letsencrypt/live/aula.lazarocardenas.edu.mx directory on your server. Generate a Diffie-Hellman group certificate. "
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
sudo mkdir -p /var/lib/letsencrypt
sudo nano /etc/cron.daily/certbot-renew
   #!/bin/sh
   certbot renew --cert-name aula.lazarocardenas.edu.mx --webroot -w /var/lib/letsencrypt/ --post-hook "systemctl reload nginx"

sudo chmod +x /etc/cron.daily/certbot-renew

```

# Configurar Nginx
````
sudo nano /etc/nginx/conf.d/moodle.conf
````
# Redirect all non-encrypted to encrypted
````
server {
    listen 80;
    listen [::]:80;
    server_name aula.lazarocardenas.edu.mx;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    
    server_name aula.lazarocardenas.edu.mx;
    root   /var/www/html/moodle;
    index  index.php;

    ssl_certificate     /etc/letsencrypt/live/aula.lazarocardenas.edu.mx/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/aula.lazarocardenas.edu.mx/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/aula.lazarocardenas.edu.mx/chain.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    access_log /var/log/nginx/moodle.access.log main;
    error_log  /var/log/nginx/moodle.error.log;
    
    client_max_body_size 25M;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ ^(.+\.php)(.*)$ {
        fastcgi_split_path_info ^(.+\.php)(.*)$;
        fastcgi_index index.php;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        include /etc/nginx/mime.types;
        include fastcgi_params;
        fastcgi_param  PATH_INFO  $fastcgi_path_info;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
    
    # Hide all dot files but allow "Well-Known URIs" as per RFC 5785
	location ~ /\.(?!well-known).* {
    	return 404;
	}
 
	# This should be after the php fpm rule and very close to the last nginx ruleset.
	# Don't allow direct access to various internal files. See MDL-69333
	location ~ (/vendor/|/node_modules/|composer\.json|/readme|/README|readme\.txt|/upgrade\.txt|db/install\.xml|/fixtures/|/behat/|phpunit\.xml|\.lock|environment\.xml) {
     	deny all;
	    return 404;
	}
}

//////////////////
sudo nginx -t

````

https://www.howtoforge.com/how-to-install-moodle-on-ubuntu-22-04/


__CRONTAB mantenimiento del script__
```
crontab -u www-data -e

 * * * * * /usr/bin/php  /var/www/html/moodle/admin/cli/adhoc_task.php --execute --keep-alive=59 >/dev/null
```

__UNOCOV para Editar PDF__
```
cd
apt install unoconv
sudo mkdir  /var/www/html/.config
chown www-data:www-data   /var/www/html/.config
sudo chmod -R 777 /var/www/html/.config
sudo chown www-data /var/www
nano /etc/php/7.4/apache2/php.ini 
    Press Ctrl and W and type "post_max_size"
    Change the value to the number of Mb you want your site to accept as uploads
    Press Ctrl and W andtype "upload_max_filesize"
    Change the value to the number of Mb you want your site to accept as uploads
    Press Ctrl and W and type "max_execution_time"
    Change the value to 600
    Press Ctrl and O
    Press Ctrl and X
 sudo service apache2 restart
 ```


___CACHE: APC within Moodle___
```
apt -y install php7.4-dev
sudo apt-get update

cd /tmp
git clone https://github.com/krakjoe/apcu
cd apcu
phpize
./configure
make
sudo make install

systemctl restart nginx
```

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

___CACHE: REDIS___
echo https://linux.how2shout.com/how-to-install-memcached-on-ubuntu-22-04-lts-server/
sudo apt install memcached libmemcached-tools
sudo systemctl status memcached
echo (ver puerto -p 11211 )
sudo apt update
sudo apt -y install redis-server
sudo systemctl enable redis-server
sudo apt -y install php-redis

sudo nano /etc/redis/redis.conf (buscar y modificar con CTR+W)
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
sudo apt -y install memcached libmemcached-tools
(sudo nano /etc/memcached.conf)
(sudo ufw allow from 192.168.100.254 to any port 11211)
sudo apt -y install php-memcached
systemctl reload apache2
```
http://localhost/moodle/cache/admin.php




# INSTALACION VIA COMMMAND LINE
```
/usr/bin/php /var/www/html/moodle/admin/cli/install
*/*/*/
.-.       
   _____                         | || |       
  /____/-.---_  .---.  .---.  .-.| || | .---. 
  | |  _   _  |/  _  \/  _  \/  _  || |/  __ \
  * | | | | | || |_| || |_| || |_| || || |___/
    |_| |_| |_|\_____/\_____/\_____||_|\_____)

Moodle 3.9.9+ (Build: 20210805) command line installation program
-------------------------------------------------------------------------------
== Choose a language ==
en - English (en)
? - Available language packs
type value, press Enter to use default value (en)
: es_mx
-------------------------------------------------------------------------------
== Permiso directorios de datos ==
valor del tipo, pulse Enter para utilizar el valor por defecto (2777)
: 
-------------------------------------------------------------------------------
== Dirección Web ==
valor del tipo
: http://aula.lazarocardenas.edu.mx/moodle
-------------------------------------------------------------------------------
== Directorio de Datos ==
valor del tipo, pulse Enter para utilizar el valor por defecto (/var/www/moodledata)
: /var/www/XXXXXXXXXXmysql
-------------------------------------------------------------------------------
== Seleccione el controlador de la base de datos ==
 mysqli 
 auroramysql 
 mariadb 
valor del tipo, pulse Enter para utilizar el valor por defecto (mysqli)
: (ENTER)
== host de la Base de Datos ==
valor del tipo, pulse Enter para utilizar el valor por defecto (localhost)
: 
-------------------------------------------------------------------------------
== Nombre de la base de datos ==
valor del tipo, pulse Enter para utilizar el valor por defecto (moodle)
: aula21
-------------------------------------------------------------------------------
== Prefijo de tablas ==
valor del tipo, pulse Enter para utilizar el valor por defecto (mdl_)
: 
-------------------------------------------------------------------------------
== Puerto de BasedeDatos ==
valor del tipo, pulse Enter para utilizar el valor por defecto ()
: 
-------------------------------------------------------------------------------
== Socket Unix ==
valor del tipo, pulse Enter para utilizar el valor por defecto ()
: 
-------------------------------------------------------------------------------
== Usuario de la base de datos ==
valor del tipo, pulse Enter para utilizar el valor por defecto (root)
: XXXXXX
-------------------------------------------------------------------------------
== Contraseña de la base de datos ==
valor del tipo
: XXXXX
-------------------------------------------------------------------------------
== Nombre completo del sitio ==
valor del tipo
: PFLC
== Nombre_de_usuario de la cuenta del administrador ==
valor del tipo, pulse Enter para utilizar el valor por defecto (admin)
: XXXXXXXXXXXXXXx
-------------------------------------------------------------------------------
== Nueva contraseña de usuario admin ==
valor del tipo
: XXXXXXXXXXXXXXX
-------------------------------------------------------------------------------
== Nuevas direcciones Email de usuario administrador ==
valor del tipo, pulse Enter para utilizar el valor por defecto ()
: XXXXXXXXXXXXXXXXx
-------------------------------------------------------------------------------
== Clave para actualización (dejar vacía para no configurarla) ==
valor del tipo
: 
-------------------------------------------------------------------------------
== Copyright ==
Moodle  - Modular Object-Oriented Dynamic Learning Environment
Copyright (C) 1999 en adelante, Martin Dougiamas (http://moodle.com)

Este programa es software libre: usted puede redistribuirlo y /o modificarlo bajo los términos de la Licencia Pública General GNU (GNU General Public License) publicada por la Fundación para el Software Libre, ya sea la versión 3 de dicha Licencia, o (a su elección) cualquier versión posterior.

Este programa se distribuye con la esperanza de que sea útil, pero SIN NINGUNA GARANTÍA; incluso sin la garantía implícita de COMERCIALIZACIÓN o IDONEIDAD PARA UN PROPÓSITO PARTICULAR.

Vea la página de información de Licencia de Moodle para más detalles: https://docs.moodle.org/en/License

¿Ha leído y comprendido los términos y condiciones?
escriba s (sí) o n (no)
: s

.....
->tinymce_spellchecker
++ Éxito ++
-->tinymce_wrap
++ Éxito ++
-->logstore_database
++ Éxito ++
-->logstore_legacy
++ Éxito ++
-->logstore_standard
++ Éxito ++
....
La instalación se completo exitosamente.


-----
```bash
# RESTAURACION DE CURSOS MUY GRANDES OBSERVACIONES
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
# CRON Cli
CRON CLI
/usr/bin/php  /var/www/html/moodle/admin/cli/cron.php
```

# GoogleChrome for Linux 
```bash
cd
sudo apt -f install
sudo dpkg --configure -a
sudo apt -f install
sudo apt-get install gdebi-core -y 
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb 
sudo gdebi google-chrome-stable_current_amd64.deb 
sudo gedit /usr/share/applications/google-chrome.desktop  
```


# PERMISOS
```bash
ECHO Moodledata segun: https://docs.moodle.org/311/en/Step-by-step_Installation_Guide_for_Ubuntu
sudo mkdir moodledata
sudo chown -R www-data:www-data moodledata/
sudo chmod -R 777 moodledata


ECHO Instalar Pluggins:
chmod -R 0777 moodle

ECHO exit
chmod -R 0755 /var/www/html/moodle
```

# ERRORS from MoodleData Cache Purge
Si purgamos el cache, no correr el script, mejor DIRECTAMENTE borrar "cache" moddledata, marca errores aveces
el problema es que se genera adentro un .php que ocupa permisos $chmod 777 -R (todo el directorio de moodledata)

# Purge caches
You can purge caches using this script:
```
 php admin/cli/purge_caches.php
 ```
# Kill all sessions
If needed for administrative reasons, you can kill all user sessions using this script:
```
php admin/cli/kill_all_sessions.php
```
As a result, all users will be logged out from Moodle.

# Reset user password
If you happen to forget your admin password (or you want to set a password for any other user on the site), you can use reset_password.php script. The script sets the correctly salted password for the given user.
```
   $ sudo -u apache /usr/bin/php admin/cli/reset_password.php
   ```
----
# Maintenance mode
To switch your site into the maintenance mode via CLI, you can use
```
   $ sudo -u apache /usr/bin/php admin/cli/maintenance.php --enable
```
To turn maintenance mode off, execute the same script with the **--disable** parameter:
```
   $ sudo -u apache /usr/bin/php admin/cli/maintenance.php --disable
```
If you don't want to enable maintenance mode immediately, but show a countdown to your users, execute the same script with the **--enablelater** parameter and the **number of minutes which the countdown** should run:
```
   $ sudo -u apache /usr/bin/php admin/cli/maintenance.php --enablelater=10
```
This script will also create and remove the climaintenance.html file for "Offline" mode.

# Schedule Tasks
php admin/cli/scheduled_task.php --list
