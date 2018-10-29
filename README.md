# Practica-04-2VM-Apache-Vagrant

## Instalar Virtualbox y vagrant
Para relizar esta práctica sera necesario instalar [Virtualbox](https://www.virtualbox.org/wiki/Downloads) y [vagrant](https://www.vagrantup.com/downloads.html) sobre el sistema operativo en el que vayamos a realizarla.

## Crear Vagrantfile para tres ubuntu server 18.04 dos server apache Apache y un server Mysql

Creamos un directorio donde contendremos las carpetas para nuestras maquinas virtuales en vagrant.
-  mkdir VMvagrant
-  cd VMvagrant
-  mkdir maquina-2web-Mysql

Desde consola nos situamos dentro de la carpeta "maquina-2web-Mysql" y lanzamos la siguientes lineas.

- vagrant init

Con esta linea de comando ya tendremos creado el archivo de configuracion "Vagrantfile" que nos permitirá configurar nuestros ubuntu server 18.04 el contendio de archivo es el siguiente:
~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"

  # Apache HTTP Server
  config.vm.define "web" do |app|
    app.vm.hostname = "web"
    config.vm.network "private_network", ip: "192.168.33.10"
    app.vm.provision "shell", path: "provision-for-apache.sh"
  end

  # Apache HTTP Server2
  config.vm.define "web2" do |app|
    app.vm.hostname = "web2"
    config.vm.network "private_network", ip: "192.168.33.11"
    app.vm.provision "shell", path: "provision-for-apache.sh"
  end

  # MySQL Server
  config.vm.define "db" do |app|
    app.vm.hostname = "db"
    config.vm.network "private_network", ip: "192.168.33.12"
    app.vm.provision "shell", path: "provision-for-mysql.sh"
  end

end
~~~

## Crear script para realizar "provision" en Vm "web" y "web2"

Dentro del archivo "Vagrantfile" definimos la ruta del script para realizar "provision" en nuestra maquina virtual. El contenido del script es el siguiente:
~~~
#!/bin/bash
apt-get update
apt-get install -y apache2
apt-get install -y php libapache2-mod-php php-mysql
sudo /etc/init.d/apache2 restart
cd /var/www/html
wget https://github.com/vrana/adminer/releases/download/v4.3.1/adminer-4.3.1-mysql.php
mv adminer-4.3.1-mysql.php adminer.php
apt-get install -y git
rm -f /var/www/html/index.html
cd /tmp
git clone https://github.com/josejuansanchez/iaw-practica-lamp.git
cp iaw-practica-lamp/src/. /var/www/html/ -R
cd /var/www/html
chown www-data:www-data * -R
apt-get install -y mysql-client-core-5.7
mysql -h 192.168.33.12 -u root -proot < /tmp/iaw-practica-lamp/db/database.sql
sed -i -e 's/localhost/192.168.33.12/' /var/www/html/config.php
sed -i -e 's/lamp_user/root/' /var/www/html/config.php
sed -i -e 's/lamp_user/root/' /var/www/html/config.php
~~~

## Crear script para realizar "provision" en Vm-"db"

Dentro del archivo "Vagrantfile" definimos la ruta del script para realizar "provision" en nuestra maquina virtual. El contenido del script es el siguiente:
~~~
#!/bin/bash
apt-get update
apt-get -y install debconf-utils

DB_ROOT_PASSWD=root
debconf-set-selections <<< "mysql-server mysql-server/root_password password $DB_ROOT_PASSWD"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $DB_ROOT_PASSWD"

apt-get install -y mysql-server
sed -i -e 's/127.0.0.1/0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
/etc/init.d/mysql restart

mysql -u root mysql -p$DB_ROOT_PASSWD <<< "GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '$DB_ROOT_PASSWD'; FLUSH PRIVILEGES;"
~~~

## Inicio de la maquina virtual "db"

- vagrant up db

## Inicio de la maquina virtual "web"

- vagrant up web

## Inicio de la maquina virtual "web"

- vagrant up web2

## Repositorio github

https://github.com/luisminics/Practica-04-2VM-Apache-Vagrant