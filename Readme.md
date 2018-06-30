# Webtrees-Docker

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
```
Then open: http://localhost:8180
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

## Cleaning up:
```bash
docker ps -a                 # show all containers
docker stop <<containr-id>>  # stop the container
docker rm <<container-id>>   # remove the container
docker network rm Webtrees-Net
docker volume rm Webtrees-Data
```

## Securing my data:
The genealogy data is kept in the MySql database container that was started. If you stop and restart this container the data remains. If you delete this container the data is gone.

To extract the genealogy data you captured in webtrees click on My page > Control panel > Family trees > Manage family trees > GEDCOM file Export > A file on your computer - continue. This will create a gedcom file that you can keep as a backup and can later import.

Media files that you upload are kept in the docker volume named Webrtrees-Data. This is mounted to the Webtrees_webserver and Webtrees_php_fpm container. These files will stay around for as long as the volume stays around. If you want to copy the images to a safe place use the `docker cp` command:

```bash
docker ps   # find the container id for either the running Webtrees_webserver or Webtrees_php_fpm container
docker cp ed98a7da6685:/var/www/data/media .  # substitute ed98a7da6685 for the id you found with the docker ps command
```



