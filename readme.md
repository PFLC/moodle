![moodlerl9.png](https://github.com/PFLC/moodle/blob/main/images/moodlerl9.png)


```bash
# Deshabilitar SELinux de forma permanente para evitar conflictos con permisos de Moodle
sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config

# Actualizar todos los paquetes del sistema para asegurar que estén al día
sudo dnf update -y

# Configurar el firewall para permitir el tráfico HTTP y HTTPS, necesario para acceder a Moodle a través de la web
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# Instalar el repositorio Remi para obtener versiones más recientes de PHP
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm

# Instalar PHP desde el repositorio Remi, Moodle 4.x requiere PHP 7.3 o superior
sudo dnf module reset php
sudo dnf module enable php:remi-8.3 -y
sudo dnf install -y php php-cli php-common

# Configurar el repositorio de MariaDB para obtener una versión compatible con Moodle 4.x, que requiere MariaDB 10.2 o superior
# MariaDB REPO WIZARD a actualizar, MOODLE requiere 10.06 o superior.
# https://mariadb.org/download/?t=repo-config&d=CentOS+Stream&v=10.11&r_m=xtom_fre
sudo tee /etc/yum.repos.d/MariaDB.repo <<EOF
# MariaDB 10.11 CentOS repository list
[mariadb]
name = MariaDB
baseurl = https://mirrors.xtom.com/mariadb/yum/10.11/centos/\$releasever/\$basearch
gpgkey = https://mirrors.xtom.com/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck = 1
EOF

# Reiniciar PHP-FPM si ya está instalado, útil para aplicar cambios en la configuración de PHP
sudo systemctl restart php-fpm

# Instalar paquetes necesarios para Moodle y sus dependencias, incluyendo servidor web, base de datos y extensiones PHP
sudo dnf install -y git mariadb mariadb-server httpd php-mysqlnd php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap curl unzip

# Habilitar y arrancar los servicios de MariaDB y Apache (httpd)
sudo systemctl enable --now mariadb httpd

# Instalar extensiones PHP adicionales requeridas por Moodle
sudo dnf install -y php-zip php-sodium php-iconv php-curl php-openssl php-tokenizer php-soap php-ctype php-zlib php-simplexml php-intl php-pecl-apcu php-json php-dom php-xmlreader php-xml php-mbstring

# Ejecutar el script de seguridad de MariaDB para establecer la contraseña del root, eliminar cuentas anónimas, deshabilitar el login root remoto y eliminar la base de datos test
sudo mariadb-secure-installation

# Configurar ajustes recomendados para Moodle en php.ini, como zona horaria, límites de memoria y configuración de OPcache
sudo sed -i 's/;date.timezone =.*/date.timezone = "UTC"/' /etc/php.ini
sudo sed -i 's/;opcache.enable=.*/opcache.enable=1/' /etc/php.ini
sudo sed -i 's/^\s*;\?\s*max_input_vars\s*=.*/max_input_vars = 5000/' /etc/php.ini

sudo sed -i 's/;opcache.memory_consumption=.*/opcache.memory_consumption=512/' /etc/php.ini
sudo sed -i 's/;opcache.max_accelerated_files=.*/opcache.max_accelerated_files=10000/' /etc/php.ini
sudo sed -i 's/;opcache.revalidate_freq=.*/opcache.revalidate_freq=2/' /etc/php.ini
sudo systemctl restart php-fpm

# Crear la base de datos de Moodle
mysql -u root -p -e "CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci; CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'password'; GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost'; FLUSH PRIVILEGES;"

# Instalar unoconv para la conversión de documentos, opcional pero recomendado para Moodle
sudo dnf install -y unoconv

# Clonar la última versión estable de Moodle desde su repositorio Git
cd /var/www/html
sudo git clone git://git.moodle.org/moodle.git
cd moodle
sudo git branch -a
sudo git branch --track MOODLE_

403_STABLE origin/MOODLE_403_STABLE
sudo git checkout MOODLE_403_STABLE

# Establecer los permisos adecuados para los directorios de Moodle
sudo chown -R apache:apache /var/www/html/moodle
sudo chmod -R 755 /var/www/html/moodle
sudo mkdir /var/www/moodledata
sudo chown -R apache:apache /var/www/moodledata
sudo chmod -R 777 /var/www/moodledata

# Instalar soporte adicional de Moodle, como aspell para la revisión ortográfica
sudo dnf --enablerepo=crb install -y aspell
```
----



```bash
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

# RESTART PHP de haber modificaciones MAX_SIZE, Etc
# Puede editar el archivo PHP.INI y cambiar a (por ejemplo) upload_max_filesize = 1000M , post_max_size = 1000M y max_execution_time = 600 .
sudo systemctl restart php-fpm
-------------

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
dnf install php-pecl-apcu -y
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

# Instalacion de Moodle 4.x via Git
cd /var/www/html
git clone git://git.moodle.org/moodle.git
cd moodle
git branch -a
git branch --track MOODLE_403_STABLE origin/MOODLE_403_STABLE 
git checkout MOODLE_403_STABLE         

# ---- Importante y puede causar frustracion los permisos -----
chown -R apache:apache /var/www/html/moodle/
chmod -R 755 /var/www/html/moodle/
mkdir /var/www/moodledata
chown -R apache:apache /var/www/moodledata
chmod -R 755 /var/www/moodledata

# Otros soportes de Moodle
dnf --enablerepo=crb install aspell

# ---- Para FavIcons buen sitio 
#[make Favicons](https://favicon.io/favicon-converter/)https://favicon.io/favicon-converter/


```
# Instalacion por Consola

```bash
cd /www/html/moodle/admin/cli
sudo php install.php
                                 .-..-.
   _____                         | || |
  /____/-.---_  .---.  .---.  .-.| || | .---.
  | |  _   _  |/  _  \/  _  \/  _  || |/  __ \
  * | | | | | || |_| || |_| || |_| || || |___/
    |_| |_| |_|\_____/\_____/\_____||_|\_____)

Moodle 4.3.3+ (Build: 20240215) command line installation program
-------------------------------------------------------------------------------
== Choose a language ==
en - English (en)
? - Available language packs
type value, press Enter to use default value (en)
:
-------------------------------------------------------------------------------
== Data directories permission ==
type value, press Enter to use default value (2777)
:
-------------------------------------------------------------------------------
== Web address ==
type value
: http://192.168.100.59/moodle
-------------------------------------------------------------------------------
== Data directory ==
type value, press Enter to use default value (/var/www/moodledata)
:
-------------------------------------------------------------------------------
== Choose database driver ==
 mysqli
 auroramysql
 mariadb
type value, press Enter to use default value (mysqli)
: mariadb
-------------------------------------------------------------------------------
== Database host ==
type value, press Enter to use default value (localhost)
:
-------------------------------------------------------------------------------
== Database name ==
type value, press Enter to use default value (moodle)
:
-------------------------------------------------------------------------------
== Tables prefix ==
type value, press Enter to use default value (mdl_)
:
-------------------------------------------------------------------------------
== Database port ==
type value, press Enter to use default value ()
:
-------------------------------------------------------------------------------
== Unix socket ==
type value, press Enter to use default value ()
:
-------------------------------------------------------------------------------
== Database user ==
type value, press Enter to use default value (root)
: moodleuser
-------------------------------------------------------------------------------
== Database password ==
type value
: !!Prepa123!!
-------------------------------------------------------------------------------
== Full site name ==
type value
: PFLC 2 Lomas Virreyes
-------------------------------------------------------------------------------
== Short name for site (eg single word) ==
type value
: PFLC2
-------------------------------------------------------------------------------
== Admin account username ==
type value, press Enter to use default value (admin)
:
-------------------------------------------------------------------------------
== New admin user password ==
type value
: !!Prepa123!!
-------------------------------------------------------------------------------
== New admin user email address ==
type value, press Enter to use default value ()
: moodle@pflc2.edu.mx
-------------------------------------------------------------------------------
== Support email address ==
type value, press Enter to use default value ()
:
-------------------------------------------------------------------------------
== Upgrade key (leave empty to not set it) ==
type value
:
-------------------------------------------------------------------------------
== Copyright notice ==
Moodle  - Modular Object-Oriented Dynamic Learning Environment
Copyright (C) 1999 onwards Martin Dougiamas (https://moodle.com)

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

See the Moodle License information page for full details: https://moodledev.io/general/license

Have you read these conditions and understood them?
type y (means yes) or n (means no)
: yes
Incorrect value, please retry
type y (means yes) or n (means no)
: y
-------------------------------------------------------------------------------
== Setting up database ==
-->System
++ install.xml: Success (24.73 seconds) ++
++ xmldb_main_install: Success (9.62 seconds) ++
++ external_update_descriptions: Success (4.27 seconds) ++

# REFERENCIAS
[https://ciq.com/blog/how-to-install-the-phpmyadmin-web-based-mysql-or-mariadb-gui-on-rocky-linux/](https://ciq.com/blog/how-to-install-the-phpmyadmin-web-based-mysql-or-mariadb-gui-on-rocky-linux/)

[https://www.cyberciti.biz/faq/disable-selinux-on-centos-7-rhel-7-fedora-linux/](https://www.cyberciti.biz/faq/disable-selinux-on-centos-7-rhel-7-fedora-linux/)

[https://docs.moodle.org/403/en/Git_for_Administrators](https://docs.moodle.org/403/en/Git_for_Administrators)

[https://webhostinggeeks.com/howto/how-to-install-lamp-on-centos-7/](https://webhostinggeeks.com/howto/how-to-install-lamp-on-centos-7/)

[https://installati.one/install-aspell-devel-rockylinux-8/](https://linux-packages.com)

https://rockylinux.pkgs.org/9/rockylinux-crb-x86_64/aspell-0.60.8-8.el9.0.1.x86_64.rpm.html


