# Full-Website-Setup

Description:
![alt slider](http://cdn.madlabbrazil.com/ex04.jpg)

[Full Presetation](https://docs.google.com/presentation/d/1DReA_GDzy6HvG0TJ1Ry-CQlmVhuNZsEbXi7VGO3-f3k/edit?usp=sharing)


#How to Install:
1. You need have installed and working docker-engine on your machine
2. Clone the project to your machine
3. On the project root just start follow down instruction copying and pasting the scripts on your shell command

##You need at leat a VPS with 2Gb of Memory!!!

###1. Create MySQL Instances
```shell-script
docker build  -t full-website-db  'https://raw.githubusercontent.com/madlabbrazil/full-website-setup/master/dockerfiles/Dockerfile-mysql' &&
docker run -d --name fllws-database-master --hostname master.madlabbrazil.com full-website-db &&
docker run -d --name fllws-database-slave --hostname slave.madlabbrazil.com --link fllws-database-master full-website-db
```
***Wait 5 min to MySQL finish some internal configurations***

```shell-script
docker exec fllws-database-master /usr/bin/curl -o /etc/mysql/my.cnf  'https://raw.githubusercontent.com/madlabbrazil/full-website-setup/master/my-master.cnf' &&
docker exec fllws-database-slave /usr/bin/curl -o /etc/mysql/my.cnf  'https://raw.githubusercontent.com/madlabbrazil/full-website-setup/master/my-slave.cnf' &&
docker restart fllws-database-master fllws-database-slave
```
***Wait 1 min to MySQL be ready!***
```shell-script
docker exec -it fllws-database-master mysql -pVoYTuebBX5srpCz -e 'CREATE USER "replication_server1"@"172.17.0.%" IDENTIFIED BY "084T92x0x0B998M"; GRANT REPLICATION SLAVE ON *.* TO "replication_server1"@"172.17.0.%";' &&
docker exec -it fllws-database-slave mysql -pVoYTuebBX5srpCz -e 'CHANGE MASTER TO MASTER_HOST = "fllws-database-master", MASTER_PORT = 3306, MASTER_USER = "replication_server1", MASTER_PASSWORD = "084T92x0x0B998M", MASTER_LOG_FILE="mysql-bin.000001", MASTER_LOG_POS=0;START SLAVE;'
```

###2. Create disk file

```shell-script
docker  create --name fllws-website-data --volume /www debian:jessie
```

###3. Create PHP containers
```shell-script
docker build  -t full-website-php5-6  'https://raw.githubusercontent.com/madlabbrazil/full-website-setup/master/dockerfiles/Dockerfile-php5-6' &&
docker  run -d --name fllws-php5-6 --volumes-from fllws-website-data --link fllws-database-master --link fllws-database-slave full-website-php5-6
```
###4. Create Nginx webserver
```shell-script
docker build -f dockerfiles/Dockerfile-nginx -t full-website-nginx . &&
docker run -d --name fllws-nginx --volumes-from fllws-website-data  --link fllws-php5-6  full-website-nginx &&
docker exec fllws-nginx /start-nginx.sh &&
docker restart fllws-nginx
```
###5. Create Varnish container
```shell-script
docker build -f dockerfiles/Dockerfile-varnish -t full-website-varnish .
docker run -d --name fllws-varnish --link fllws-nginx -p 80:80 --volumes-from fllws-website-data --env 'VCL_CONFIG=/default.vlc' full-website-varnish
```


Informativos:
O IP do varnish para se colocado no seu navegador pode ser descoberto assim:

```shell-script
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' fllws-varnish
```
