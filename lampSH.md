# Se aplica TeddySUN LAMPSERVER para Rocky Linux 9
```
- Habilitar SODIUM  extensiones de php
- PHP 8
```

# Basado parcialmente en https://idroot.us/install-moodle-rocky-linux-9/

```
cd /tmp
git clone git://git.moodle.org/moodle.git
cd moodle
sudo git branch -a
sudo git branch --track MOODLE_42_STABLE origin/MOODLE_402_STABLE
sudo git checkout MOODLE_402_STABLE
sudo cp -R /tmp/moodle /data/www/default
sudo mkdir /data/www/moodledata23
sudo chown -R apache /data/www/moodledata23
sudo chmod -R 777 /data/www/moodledata23
sudo chmod -R 777 /data/www/default/moodle
cd /data/www/default

crontab -u apache -e
* * * * * /usr/bin/php /data/www/default/moodle/admin/cli/adhoc_task.php --execute --keep-alive=59 >/dev/null 2>&1
*/15 * * * * /usr/bin/php  /data/www/default/moodle/admin/cli/cron.php  >/dev/null 2>&1

```

# Modificaciones en PHP.ini
sudo nano /etc/php.ini
```
date.timezone = UTC
    upload_max_filesize = 256M
    post_max_size = 256M
    max_input_vars = 5000 (SE AGREGA)
        
```

# Mysql crear dB y usuario
```
sudo mysql -p

   CREATE DATABASE moodledb DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    create user 'moodleuser'@'localhost' IDENTIFIED BY 'YourPassword23!';
    GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodledb.* TO 'moodleuser'@'localhost';
  FLUSH PRIVILEGES;
  exit;
   
 ```
  ```
 Apache: 
Default Website: http://187.191.62.164
Web root location 	/data/www/default
Database: mysql-8
MySQL Location: /usr/local/mysql
MySQL Data Location: /usr/local/mysql/data
MySQL Root Password: ***
```
# UNOCONV para PDF editarlos
```
sudo dnf install libreoffice libreoffice-pyuno
pip install unoconv
nano prueba.txt
  poner algo para probarlo y guardar
unoconv -f pdf prueba.txt
ls
```

# Install ClamAV on Rocky Linux 9
```
sudo dnf check-update
sudo dnf install dnf-utils
sudo dnf install epel-release
sudo dnf install clamav clamd clamav-update
sudo setsebool -P antivirus_can_scan_system 1
sudo freshclam
sudo systemctl start clamav-freshclam
sudo systemctl status clamav-freshclam



```

# REDIS via https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-rocky-linux-9
```
sudo dnf install redis nano
echo "passworddeREDIS" | sha1sum
sudo nano /etc/redis/redis.conf
    supervised systemd
    requirepass PONERPASSWORD
sudo systemctl start redis.service
sudo systemctl enable redis
redis-cli ping
  Output
  PONG
sudo systemctl restart redis
redis-cli
   auth  PONERPASSWORD
   set key1 10
   get key1
   quit

  
------- Pendiente
 
 # Instalar Moodle 4.2 via https://www.howtoforge.com/how-to-install-moodle-elearning-platform-on-rocky-linux-8/
 ```
 sudo mkdir /data/www/default/moodle
 sudo chown -R $USER:$USER /data/www/default/moodle
 cd /data/www/default/moodle
 git clone https://github.com/moodle/moodle.git .
git branch -a 
git branch --track MOODLE_400_STABLE origin/MOODLE_402_STABLE
git checkout MOODLE_402_STABLE
sudo mkdir /data/www/moodledata23
sudo chmod -R 775 /data/www/moodledata
----DUDA -----sudo chmod -R apache /data/www/moodledata


# DOCKER
# Basado en https://docs.rockylinux.org/gemstones/docker/
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl --now enable docker


------
# HTOP para monitoreo de recursos
# Installation epel source (also called repository)
```
sudo dnf -y install epel-release
# Generate cache
sudo dnf makecache
# Install htop
sudo dnf -y install htop
```
# Install SSL via https://www.howtoforge.com/how-to-install-moodle-elearning-platform-on-rocky-linux-8/#step-8---install-ssl
```
sudo /etc/init.d/httpd stop
sudo dnf install certbot
sudo certbot certonly --standalone --agree-tos --no-eff-email --staple-ocsp --preferred-challenges http -m name@______.com -d moodle._____.com
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
sudo nano /etc/cron.daily/certbot-renew
///Copiar Archivo///
#!/bin/sh
certbot renew --cert-name aula.lazarocardenas.edu.mx --webroot -w /var/lib/letsencrypt/ --post-hook "/etc/init.d/httpd reload"

sudo chmod +x /etc/cron.daily/certbot-renew
/etc/init.d/httpd  start

echo "Falta modificar el CONFIG.PHP para que se refleje el https://aula..."

```
# Testing https:

https://letsdebug.net



