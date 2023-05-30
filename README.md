# MOODLE 3.9+ Ubuntu 20
The perfect Moodle, by Rene Solis
NOTA: Este moodle tube que invertir tiempo pues docentes de ciencias basicas requieren entregar observaciones en PDF editado con tableta o pizarra. Unoconv hubo diversos problemas, desde un libreoffice pluggins hasta la obsolecencia del script.

UBUNTU amigable para Unoconv con permisos

# SETUP
__Perfect Server HOWTO:Secure y tiempo__
```
service apparmor stop
update-rc.d -f apparmor remove 
apt-get remove apparmor apparmor-utils
```

__Perfect Server HOWTO:Ambiente__
```
sudo apt -y  install apache2 mysql-client mysql-server php libapache2-mod-php ntp
dpkg-reconfigure dash
```
__Base de Datos__
```
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
```
__PHP7 most compatible (tested 8 and 9 fail)__
```
sudo apt -y install graphviz aspell ghostscript clamav php7.4-pspell php7.4-curl php7.4-gd php7.4-intl php7.4-mysql php7.4-xml php7.4-xmlrpc php7.4-ldap php7.4-zip php7.4-soap php7.4-mbstring

sudo service apache2 restart

sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
__Search CTR+W  in nano editor (SIN LINEAS EN BLANCO)__
```
/// Scroll down to the [mysqld] section and under Basic Settings add the following line under the last statement.///// SIN LINEAS EN BLANCO
default_storage_engine = innodb
innodb_file_per_table = 1
     #innodb_file_format = Barracuda (MARCA ERROR)
/// Salvar con CTR+O-y-CTR+X salvar y salir.

sudo service mysql restart
```
__SQL ajustes del DBA y BD (requiere contrasena)__
```
sudo mysql -u root -p 
CREATE DATABASE ponernombre DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
use ponernombre;
CREATE USER 'ponernombre'@'%' IDENTIFIED BY 'Escuelita123';
GRANT ALL PRIVILEGES ON basededatos.* TO 'ponernombre'@'%' WITH GRANT OPTION;
flush privileges;
quit
```

__GIT Traer la version de MOODLE 3.9+ a instalar__
```
sudo apt -y install git
cd /tmp
git clone git://git.moodle.org/moodle.git
cd moodle
sudo git branch -a
sudo git branch --track MOODLE_39_STABLE origin/MOODLE_39_STABLE
sudo git checkout MOODLE_39_STABLE
sudo cp -R /tmp/moodle /var/www/html/
sudo mkdir /var/www/moodledata21
sudo chown -R www-data /var/www/moodledata21
sudo chmod -R 777 /var/www/moodledata21
sudo chmod -R 777 /var/www/html/moodle
cd /var/www/html
```
(NOTA Permisos para Cerrar es ***sudo chmod -R 0755 /var/www/html/moodle***)

__CRONTAB mantenimiento del script__
```
crontab -u www-data -e

 * * * * * /usr/bin/php  /var/www/html/moodle/admin/cli/adhoc_task.php --execute --keep-alive=59 >/dev/null
```
_Moodle 4 pide un cambio en PHP se dice "max_input_vars=5000"
```
nano sudo nano /etc/php/7.4/apache2/php.ini 
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

sudo service apache2 reload
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
REDIS https://linuxize.com/post/how-to-install-memcached-on-ubuntu-20-04/
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
