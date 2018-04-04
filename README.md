## PHP 7.2 (php-fpm) + MongoDb project docker Image

This Docker image is based on [Official Yii2 docker image](https://github.com/yiisoft/yii2-docker), but uses only alpine version of php container and has a little another structure.

Inspired by [avb-89's article](https://habrahabr.ru/post/349704/)
 
### Structure

```
app 
data
-db
nginx
-etc/nginx/conf.d/nginx.conf
-usr/share/nginx/html
-var/log/nginx
php
-image-files
--root/.bashrc
--usr/local
---bin
----composer
----docker-php-entrypoint
---etc
----php
-----php.ini
-----conf.d
------mongodb.ini
------xdebug.ini
-Dockerfile
tests
--requirements
---views
----console
-----index.php
----web
-----css.php
-----index.php
---requirements.php
---YiiRequirementChecker.php
--requirements.php
.env-dist
.gitignore
docker-compose.yml
README.md
```

**app** folder is where your yii application should be placed

**nginx/etc/nginx/conf.d/nginx.conf** is nginx config

**php/image-files/usr/local/etc/php/php.ini** is php config

**.env** file is environment for docker-compose (.env-dist is default config to copy to .env. See [official documentation](https://github.com/yiisoft/yii2-docker))

### Create project database

1. Connect to mongodb container

```
sudo docker exec -it mongodb /bin/bash
```

2. Connect to mongodb with user and password specified in .env

```
mongo -u "mongousradmin" -p "mongopassadmin" -authenticationDatabase "admin"
```

3. create database and user as usual

```
 use sitebase
 db.createUser(
   {
     user: "siteusr",
     pwd: "sitepass",
     roles: [ { role: "readWrite", db: "sitebase" },
              { role: "read", db: "reporting" } ]
   }
 )
```

4. To connect to mongodb as new user type in mongodb container console

```
mongo -u "siteusr" -p "sitepass" -authenticationDatabase "sitebase"
```

### Yii2 Specific

1. Connect to php-fpm container's bash and run following command

```
composer create-project yiisoft/yii2-app-basic /app
```

Note that /app folder must be empty

### Docker commands

1. To build an image run following command

```
docker-compose build
```

2. To connect running php container's bash

```
docker-compose exec -i -t CONTAINER_NAME /bin/bash
```

3. To list existing containers

```
docker container ls
```

4. Stop and remove all docker containers

```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

5. Prune all containers, images, volumes and networks

```
docker system prune -a
```

6. Rebuild specified service

```
docker-compose up -d --no-deps --build <service_name>
```