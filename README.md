# ZabbixContainer
Using Docker Containers for Zabbix Monitoring by using MicroServices Approach 

***

## Pull base images
```
$ docker pull mysql/mysql-server
$ docker pull zabbix/zabbix-server-mysql
$ docker pull zabbix/zabbix-web-nginx-mysql
$ docker pull zabbix/zabbix-agent
```
***

## Zabbix Mysql Database
```
$ docker run --name=ZabbixDB -p 3306:3306 -d mysql/mysql-server
```

```
$ DBUSER="zabbix"
$ DBPASS="ufuk1023" 
$ DBROOTPASS=$(docker logs ZabbixDB 2>&1 | grep GENERATED | awk {'print $5'} )
```

```
$ docker exec -it ZabbixDB mysql -uroot -p$DBROOTPASS 
```
```
$ ALTER USER 'root'@'localhost' IDENTIFIED BY 'ufuk1023';
$ create database zabbix character set utf8;
$ grant all privileges on zabbix.* to zabbix@'%' identified by 'ufuk1023';
$ flush privileges;
$ quit;
```

```DBHOST=$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' ZabbixDB)```

***

## Zabbix Server

```docker run --name ZabbixServer -p 10051:10051 -e DB_SERVER_HOST=$DBHOST -e MYSQL_USER=$DBUSER -e MYSQL_PASSWORD=$DBPASS -d zabbix/zabbix-server-mysql```

```ZSHOST=$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' ZabbixServer)```

```
#> docker exec -it ZabbixServer /bin/bash
#> ps -ef
```

***

## Nginx Web Server

```docker run --name ZabbixWeb -p 80:80 -p 443:443 -e DB_SERVER_HOST=$DBHOST -e MYSQL_USER=$DBUSER -e MYSQL_PASSWORD=$DBPASS -e ZBX_SERVER_HOST=$ZSHOST -e PHP_TZ="Europe/Istanbul" -d zabbix/zabbix-web-nginx-mysql```

```
#> docker logs ZabbixWeb
#> docker ps -a
```

```WEBHOST=$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' ZabbixWeb)```

```
#> Test link by using following link
#> elinks http://$WEBHOST/zabbix
```
***

## Zabbix Agent

```docker run --name ZabbixAgent1 -p 10050:10050 -e ZBX_HOSTNAME="agent1" -e ZBX_SERVER_HOST=$ZSHOST  -d zabbix/zabbix-agent```

```
#> docker logs ZabbixAgent1
```

```ZA1HOST=$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' ZabbixAgent1)```
