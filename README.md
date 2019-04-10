
# Debian Install
Install instructions and configurations for _my devops environment_ based on debian , including : nextcloud, mariadb, docker, nextcloud, gitlab etc.

## DEBIAN 9.8

#### add debian mirrors in _/etc/apt/sources.list_

```
deb http://mirrors.ustc.edu.cn/debian/ stretch main
deb-src http://mirrors.ustc.edu.cn/debian/ stretch main

deb http://security.debian.org/debian-security stretch/updates main
deb-src http://security.debian.org/debian-security stretch/updates main

# stretch-updates, previously known as 'volatile'
deb http://mirrors.ustc.edu.cn/debian/ stretch-updates main
deb-src http://mirrors.ustc.edu.cn/debian/ stretch-updates main

```
#### Install basic packages
```bash
su
apt-get update
apt-get install sudo aptitude open-jdk8 apache2 mariadb-server postgres git docker docker-compose 
```
#### add current user to /etc/sudoer:
```
liang   ALL=(ALL:ALL) ALL
```

------

## VS Code

#### Download VS code :
``` 
sudo dpkg -i code_1.33.0-1554390824_amd64.deb
sudo update-alternatives --set editor /usr/bin/code
```

------

## Gitlab
#### install
```bash
sudo apt-getisntall libapache2-mod-php7.0 sud php7.0-gd php7.0-json php7.0-mysql php7.0-curl php7.0-mbstring php7.0-intl php-imagick php7.0-xml php7.0-zip
sudo gitlab-ctl reconfigure
systemctl restart gitlab-ce
sudo nano /etc/apache2/sites-enabled/gitlab.conf
```
#### default data is store in /var/opt/gitlab/git-data
```json
git_data_dirs({ "default" => { "path" => "/mnt/nas/git-data" } })
```

------

## MariaDB

#### start mariadb
```shell
sudo systemctl start mariadb
```
#### set root password :
```bash
sudo mysql_secure_installation
```
```
mysql -u root -p
```
#### Create a nextcloud DB
```sql
CREATE DATABASE nextcloud ;
CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'nextcloud';
GRANT ALL PRIVILEGES ON * . * TO 'nextcloud'@'localhost'; 
FLUSH PRIVILEGES;
```

#### trouble-shooting

if cannot login, su with root and  login without password, run : 
```sql
SELECT user, host, plugin FROM mysql.user;
```
If you see unix_socket in the plugin column, that's the reason.
To return to the usual password authentication, run :
```sql
UPDATE mysql.user SET plugin = '' WHERE plugin = 'unix_socket';
FLUSH PRIVILEGES;
```

#### create a empty db
```shell
systemctl stop mariadb
rm -R /var/lib/mysql/*
mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
systemctl start mariadb
```

-------

## Apache2

#### config nextcloud host
sudo nano /etc/apache2/apache2.conf 

```
Alias /nextcloud /var/www/nextcloud
<Directory /var/www/nextcloudo>
  Options +FollowSymlinks
  AllowOverride All

 <IfModule mod_dav.c>
  Dav off
 </IfModule>

 SetEnv HOME /var/www/nextcloud
 SetEnv HTTP_HOME /var/www/nextcloud

</Directory>
```
#### start apache
```
sudo systemctl start apache2
```

-------

## NextCloud

#### add access right
```
sudo chown -R www-data:www-data /var/www/nextcloud/
```
#### change data folder
```
sudo mkdir /home/data/nextcloud
sudo chown -R www-data:www-data /home/data/nextcloud

sudo a2enmod proxy rewrite headers env mime
```

#### trouble-shooting

If internal error when opening http://localhost/nextcloud
need run the :`sudo chown -R www-data:www-data /home/data/nextcloud`command 

#### reset password :
```
sudo -u www-data php /var/www/nextcloud/occ user:resetpassword admin
```

#### system-surveillance-tool
top,free,w,vmstate,ps,uptime....

https://www.cyberciti.biz/tips/top-linux-monitoring-tools.html


-------

## Docker & Docker-Compose

#### docs links
https://docs.docker.com/install/linux/docker-ce/debian/

#### Download docker deb : 
https://download.docker.com/linux/debian/dists/

```
sudo dpkg -i /path/to/package.deb
```
#### change source
sudo nano /etc/docker/daemon.json
```json
{
	"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```
```
systemctl restart docker.service

sudo docker-compose up -d
```

#### create docker-compose.yml file for mariadb + nextcloud
```yaml
version: '2'

services:
  db:
    image: mariadb
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    restart: always
    volumes:
      - ./mysql:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_PASSWORD=nextcloud
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  app:
    image: nextcloud
    ports:
      - 8888:80
    links:
      - db
    volumes:
      - ./nextcloud:/var/www/html
    restart: always
```

-------

## Expose SSH connexion
```
sudo nano /etc/ssh/sshd_config
```

```
+ PermitRootLogin yes
```
```
sudo systemctl restart ssh
```

-------

## Active Samba Share
```bash
apt install libcups2 samba samba-common cups

nano /etc/samba/smb.conf
```

````conf
[global]
workgroup = WORKGROUP
server string = Samba Server %v
netbios name = debian
security = user
map to guest = bad user
dns proxy = no
 

[share]
path = /home/liang/share/
browsable = yes
writable = yes
guest ok = yes
read only = no

[photo]
path = /home/photo/
browsable = yes
writable = yes
guest ok = no
read only = no 

[film]
path = /home/film/
browsable = yes
writable = yes
guest ok = no
read only = no 


````

## NodeJS

#### install
```bash
curl -sL https://deb.nodesource.com/setup_11.x | bash -

sudo apt-get install -y nodejs gcc g++ make
```
#### add npm mirror
```
npm config set registry https://registry.npm.taobao.org
```




 

