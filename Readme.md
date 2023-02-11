# Webtrees-Docker

## Abstract

Webtrees is an open source web server application that renders genealogy data from a GEDCOM file in a nice navigatable web site. The data can be edited and pictures can be added. Check out their website.

## Up and running in 5 minutes:

```bash
git clone git@github.com:richardeigenmann/Webtrees-Docker.git
cd Webtrees-Docker
docker network create Webtrees-Net
docker volume create Webtrees-Data
# THINK: do you want default passwords on the db?
# if not, modify them in the file docker-compose.yml
# under 'environmnet'!
docker-compose up -d --build

# If you get an error about the permissions log into the container and correct them
docker exec -it Webtrees_webserver bash                            
root@827bbf76cde7:/# chmod 777 /var/www/data
```

Then open: http://localhost

The Database paramters are:

- Server name: Webtrees_mysql_db
- Port number: 3306
- Database user account: wt
- Database password: password

Database and table names:

- Database name: webtrees
- Table prefix: leave unchaged

Administrator account

- Your Name: Bob Smith
- Username: bob
- Password: password
- E-mail address: nobody@gmail.com

Now head over to: https://www.webtrees.net
In particular to: https://wiki.webtrees.net/en/Users_Guide

## Cleaning up everything:

```bash
docker-compose down --rmi all --volumes --remove-orphans
# docker stop Webtrees_webserver
# docker stop Webtrees_php_fpm
# docker stop Webtrees_mysql_db
# docker rm Webtrees_webserver
# docker rm Webtrees_php_fpm
# docker rm Webtrees_mysql_db
# docker network rm Webtrees-Net
# docker volume rm Webtrees-Data
```

## Securing my data:

The genealogy data is kept in the MySql database container that was started. If you stop and restart this container the data remains. If you delete this container the data is gone.

To extract the genealogy data you captured in webtrees click on My page > Control panel > Family trees > Manage family trees > GEDCOM file Export > A file on your computer - continue. This will create a gedcom file that you can keep as a backup and can later import.

Media files that you upload are kept in the docker volume named Webrtrees-Data. This is mounted to the Webtrees_webserver and Webtrees_php_fpm container. These files will stay around for as long as the volume stays around. If you want to copy the images to a safe place use the `docker cp` command:

```bash
docker cp Webtrees_webserver:/var/www/data/media .  
```

## How it works

The docker-compose.yml file declares the 3 containers that are used. The 3 containers will connect with each other over the Docker internal network Webtrees-Net.

The configuration data for the Webtrees app and the media files will reside on the Webtrees-Data volume.

### Webtrees_webserver

This container is defined by the Dockerfile (of the same name) and inherits from nginx. It receives a customised vhost.conf which forwards requests for php scripts to the fcgi container.

The nginx.conf is modified to allow image uploads up to 40M as the default is unusable.

The container is then populated with the webtrees code.

The startup starts fcgiwrap and then nginx in non daemon mode.

The webserver will run on port 8180. Change that to whatever you like in the docker-compose.yml and vhost.conf file.

### Webtrees_php_fpm

The php scripts are run from a separate container. This inherits from php:7-fpm. Fastcgi acts as glue between the nginx container and the fpm container communicating over port 9000 on a private Docker network. 

The Dockerfile-php adds libpng, freetype, libjpeg, libxml2 and unzip to the container. It also installs mysqli, pdo, pdo_mysql and xml as well as gd.

Webtrees is intalled into both containers so that the web container has the static images and the php container has the scripts.

### Webtrees_mysql_db

The database uses the mysql image from Dockerhub as-is.

The docker-compose.yml file lists the username and password that will be used when the server starts up. You are advised to change this to something secure.

The database is not visible to the host and tools like phpmysqladmin will not find it. Add a port mapping to the docker-compose.yml file if you want to connect to the db.

## Interacting with the running containers

The below command lists all running containers.

```bash
docker ps
```

You can connect to a running container and use the shell to check things:

```bash
docker exec -it Webtrees_webserver bash
docker exec -it Webtrees_php_fpm bash
```

For instance the following command shows the contents of the webserver directory:

```bash
ls /var/www
```

The webserver logs are redirected to stdout and stderr of the container and can be viewed with

```bash
docker logs Webtrees_webserver
```

You can interact with the database
```bash
docker exec -it Webtrees_mysql_db bash
mysql -u root -p
```

## Setting up Let's encrypt

The server needs to be on the Internet and port 80 and 443 need to be visible.

```bash
docker exec -it Webtrees_webserver bash
apt-get update && apt-get install --assume-yes certbot python3-certbot-nginx
certbot --nginx -d www.example.com
nginx -t && nginx -s reload
```

