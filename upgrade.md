# para actualizar Moodle 4.x
```
sudo -u apache /usr/bin/php /data/www/default/moodle/admin/cli/maintenance.php  --enablelater=60
cd  /data/www/default

sudo echo "Bajar la nueva version"
sudo mv moodle moodle.backup
sudo wget https://download.moodle.org/download.php/direct/stable402/moodle-4.2.1.tgz
sudo tar xvzf moodle-4.2.1.tgz
sudo rm moodle-4.2.1.*

echo "Copiar la configuracion"
sudo cp moodle.backup/config.php moodle
echo "sudo cp -pr moodle.backup/theme/mytheme moodle/theme/mytheme"
echo "sudo cp -pr moodle.backup/mod/mymod moodle/mod/mymod"

sudo chown -R root:root moodle

echo "Para instalar pluggins y actualizaciones"
sudo chmod -R 777 moodle

echo "Actualizar"
sudo -u apache /usr/bin/php /data/www/default/moodle/admin/cli/upgrade.php
sudo -u apache /usr/bin/php /data/www/default/moodle/admin/cli/maintenance.php  --disable

echo "Acceder HTTP y aceptar la actualizacion"
echo "Cerrar el directorio solo lectura"
echo "Aqui aveces se queda en un loope de actualizacion, reiniciar y reintentar"
echo " No olvidar de cerrarlo"
sudo chmod -R 755 moodle
```

# Asegurar SSH via https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-rocky-linux-9#step-3-%E2%80%94-logging-in-to-your-rocky-linux-server-using-ssh-keys

