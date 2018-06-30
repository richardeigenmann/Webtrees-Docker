# Webtrees-Docker

## Up and running in 5 minutes:

```bash
git clone git@github.com:richardeigenmann/Webtrees-Docker.git
cd Webtrees-Docker
docker network create Webtrees-Net
docker volume create Webtrees-Data
# THINK: do you want default passwords on the db?
# if not, modify them in file docker-compose.yml 
# under 'environmnet'!
docker-compose up -d --build
```
Then open: http://localhost:8180

## Cleaning up:
```bash
docker ps -a                 # show all containers
docker stop <<containr-id>>  # stop the container
docker rm <<container-id>>   # remove the container
docker network rm Webtrees-Net
docker volume rm Webtrees-Data
```