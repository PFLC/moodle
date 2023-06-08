# Se aplica TeddySUN LAMPSERVER para Rocky Linux X
```
- Habilitar SODIUM  extensiones de php
```

# Con PHP8

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
    * * * * * /usr/bin/php  /data/www/default/moodle/admin/cli/adhoc_task.php --execute --keep-alive=59 >/dev/null
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
 mysql > CREATE DATABASE moodledb DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
 mysql >create user 'moodleuser'@'localhost' IDENTIFIED BY '......passwrod....';
 mysql >GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,CREATE TEMPORARY TABLES,DROP,INDEX,ALTER ON moodledb.* TO 'moodleuser'@'localhost';
 mysql > FLUSH PRIVILEGES;
 mysql >exit
 ```
 
