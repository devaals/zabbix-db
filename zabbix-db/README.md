Zabbix NIIAS
==========
This Image based on zabbix-db-mariadb of MonitoringArtist.
[Zabbix NIIAS](https://github.com/devaals/zabbix-db) is a standard Zabbix prepared for Docker world. You must install Zabbix package (rpm, deb, ...) in the old world. Similarly, you need to pull Zabbix Docker image in the Docker world. This Docker image contains standard Zabbix + additional XXL (community) extensions. Routine tasks such as import of Zabbix DB are automated, so it's prepared for easy deployment.


Zabbix DB - MariaDB 10.0
========================

This is a MariaDB 10.0 Docker. Based on [zabbix/zabbix-db-mariadb](https://hub.docker.com/r/monitoringartist/zabbix-db-mariadb/) 
image. Built on top of official [centos:centos7](https://registry.hub.docker.com/_/centos/) 
image. Inspired by [Tutum](https://github.com/tutumcloud)'s 
[tutum/mariadb](https://github.com/tutumcloud/tutum-docker-mariadb) image.

Note: be aware that, by default MariaDB in this container is configured to use 
1GB memory (innodb_buffer_pool_size in 
[tuning.cnf](container-files/etc/my.cnf.d/tuning.cnf)). If you try to run it on 
node with less memory, it will fail.

## Usage

Run the image as daemon and bind it to port 3306:

```  
docker run \
	-t \
	-v /opt/mysql:/var/lib/mesql \
    	-v /etc/localtime:/etc/localtime:ro
	--name zabbix-db \
	-p 3306:3306 \
	--env="MARIADB_USER=zabbix" \
	--env="MARIADB_PASS=zabbix" \
	--env="DB_innodb_buffer_pool_size=512M" \
	devaal/zabbix-db:latest
```  

## Environmental variables
You can use environmental variables to config MariaDB. Available variables:


| Variable | Default value |
| -------- | ------------- |
|DB_max_allowed_packet | 96M |
|DB_open_files_limit | 4096 |
|DB_max_connections | 200 |
|DB_query_cache_size | 0 |
|DB_query_cache_type | 0 |
|DB_sync_binlog | 0 |
|DB_innodb_buffer_pool_size | 512M |
|DB_innodb_log_file_size | 128M |
|DB_innodb_flush_method | O_DIRECT |
|DB_innodb_old_blocks_time | 1000 |
|DB_innodb_flush_log_at_trx_commit | 0 |

## Configuration from volume`s:
Full config files can be also used. Environment configs will be overriden by 
values from config files in this case. You need only to add 
*/etc/custom-config/* volume:

```
-v /host/custom-config/:/etc/custom-config/
```

##Configuration time & timezone:
```
-v /etc/localtime:/etc/localtime:ro
```

##Configuration database volume:
```
-v /opt/mysql:/var/lib/mysql
```

Available files:

| File | Description |
| ---- | ----------- |
| mariadb-tuning.cnf | MariaDB configuration file |

The first time that you run your container, a new user admin with all privileges 
will be created in MariaDB with a random password. To get the password, check 
the logs of the container by running:  
`docker logs <CONTAINER_ID>`  

You will see an output like the following:

```
	========================================================================
    You can now connect to this MariaDB Server using:

        mysql -uadmin -pCoFlnc3ZBS58 -h<host>

    Please remember to change the above password as soon as possible!       
    MariaDB user 'root' has no password but only allows local connections
    ========================================================================
```  
In this case, `CoFlnc3ZBS58` is the password generated for the `admin` user.

### Custom Password for user admin 
If you want to use a preset password instead of a random generated one, you can 
set the environment variable MARIADB_PASS to your specific password when 
running the container:  

`docker run -d -p 3306:3306 -e MARIADB_PASS="mypass" devaal/zabbix-db`

### Mounting the database file volume from other containers
One way to persist the database data is to store database files in another 
container. To do so, first create a container that holds database files:  

`docker run -d -v /var/lib/mysql --name db-data busybox:latest`  

This will create a new container and use its folder `/var/lib/mysql` to store 
MariaDB database files. You can specify any name of the container by using 
`--name` option, which will be used in next step.

After this you can start your MariaDB image using volumes in the container 
created above (put the name of container in `--volumes-from`).  

`docker run -d --volumes-from db-data -p 3306:3306 devaal/zabbix-db:latest`
