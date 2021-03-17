# MOODLE 3.9+ Ubuntu 20
The perfect Moodle, by Rene Solis


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
