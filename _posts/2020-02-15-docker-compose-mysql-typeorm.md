---
title: "Access denied for user Error"
author: bkelley
layout: post
date: 2019-02-15 12:45:42 -0600
categories: posts
image: /assets/posts/docker-compose-mysql.png
description: >
  Error - ER_ACCESS_DENIED_ERROR: Access denied for user 'admin'@'localhost' (using password: YES)
---

- [The Problem](#the-problem)
  - [What Works](#what-works)
    - [PHPMyAdmin](#phpmyadmin)
    - [MySQL Workbench](#mysql-workbench)
    - [Docker Exec](#docker-exec)
  - [What Doesn't Work](#what-doesn't-work)
    - [MySQL Command Line](#mysql-command-line)
    - [Node.js Driver For MySQL](#node.js-driver-for-mysql)
  - [Attempts To Fix](#attempts-to-fix)
    - [docker-compose rm -v](#docker-compose-rm--v)
    - [Validating Docker Container](#validating-docker-container)
    - [Changing Docker Port](#changing-docker-port)
    - [Explicit Permissions](#explicit-permissions)
- [The Solution](#the-solution)

## The Problem

For my [Horsin' Around](https://github.com/Kelley12/horsin-around) application I have a [docker-compose](https://docs.docker.com/compose/) to spin up docker containers for [MySQL](https://www.mysql.com/) and [PHPMyAdmin](https://www.phpmyadmin.net/). The yaml configuration file can be found [here](https://github.com/Kelley12/horsin-around/blob/develop/docker-compose.yml). These containers start up and work fine with a simple `docker-compose up` command.

### What Works

#### PHPMyAdmin

I can connect to PHPMyAdmin using the root user (Username: `root`, Password: `root`). I can see that the horsin-around database was created successfully. The user `admin` is created and given full permissions to the `horsin-around` database from any host (signified by `%`).

#### MySQL Workbench

I can connect to the `horsin-around` database with MySQL Workbench with this configuration

- Host: `localhost`
- Port: `3306`
- User: `admin`
- Password: `admin`

#### Docker Exec

I am also able to connect to the docker image and login to mysql with this command

```bash
docker exec -it mysql mysql --user=admin --password=admin
```

Then I was able to run `SELECT USER(),CURRENT_USER();` and `SHOW GRANTS;` and get the expected output

```bash
mysql> SELECT USER(),CURRENT_USER();
+-----------------+----------------+
| USER()          | CURRENT_USER() |
+-----------------+----------------+
| admin@localhost | admin@%        |
+-----------------+----------------+
1 row in set (0.00 sec)

mysql> SHOW GRANTS;
+----------------------------------------------------------+
| Grants for admin@%                                       |
+----------------------------------------------------------+
| GRANT USAGE ON *.* TO 'admin'@'%'                        |
| GRANT ALL PRIVILEGES ON `horsin-around`.* TO 'admin'@'%' |
+----------------------------------------------------------+
2 rows in set (0.00 sec)
```

### What Doesn't Work

There are a lot of things that do work which makes the things that don't work that much more confusing.

#### MySQL Command Line

I am unable to connect to connect using command line, I have tried all sorts of variations

```bash
mysql --user=admin --password=admin
mysql --user=admin --password=admin --database=horsin-around
mysql --user=admin --password=admin --database=horsin-around --port=3306
mysql --host=127.0.0.1 --user=admin --password=admin --database=horsin-around --port=3306
mysql --host=localhost --user=admin --password=admin --database=horsin-around --port=3306
```

But to no avail, each results in `ERROR 1045 (28000): Access denied for user 'admin'@'localhost' (using password: YES)`

I also tried specifying the protocol (tcp and socket)

```bash
mysql --host=localhost --user=admin --password=admin --database=horsin-around --port=3306 --protocol=tcp
mysql --host=localhost --user=admin --password=admin --database=horsin-around --port=3306 --protocol=socket
```

The `socket` protocol gave me the same error as above, the `tcp` protocol gave me this error

```bash
ERROR 2026 (HY000): SSL connection error: error:00000001:lib(0):func(0):reason(1)
```

I also tried changing `localhost` to `127.0.0.1`, when using protocol of `socket` it resulted in this error

```bash
ERROR 2047 (HY000): Wrong or unknown protocol
```

Changing the protocol to `tcp` resulted in the `Error 1045` above.

#### Node.js Driver For MySQL

The main problem is that when the application tries to connect to MySQL, it gets the following error

`Error - ER_ACCESS_DENIED_ERROR: Access denied for user 'admin'@'localhost' (using password: YES)`

I used a `console.log` to print the TypeORM `ConnectionOptions` and got the configuration I would expect

```javascript
{
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  database: 'horsin-around',
  username: 'admin',
  password: 'admin',
  synchronize: true,
  logging: true,
  dropSchema: false,
  ...
}
```

These are the same configuration options that I used to connect with MySQL Workbench and using the MySQL command line client

```bash
mysql --host=localhost --user admin --password=admin --database=horsin-around --port=3306
```

### Attempts To Fix

There was a lot of Googling, trying this and trying that, tweaking and testing. These are some of the thigns I tried (and remembered to document).

### docker-compose rm -v

I have tried deleting the volume folder and removing the docker images as suggested in [this mysql docker-library issue](https://github.com/docker-library/mysql/issues/51#issuecomment-76989402)

```bash
docker-compose rm -v
```

#### Validating Docker Container

I used a `docker ps` to look at the processes to see the host IP and ports

```bash
CONTAINER ID        IMAGE                   COMMAND                  CREATED              STATUS              PORTS      
              NAMES
ded1b34bde46        phpmyadmin/phpmyadmin   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:808
0->80/tcp     phpmyadmin
046254e59a92        mysql:5.7.21            "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:330
6->3306/tcp   mysql
```

I tried chaning the host to `0.0.0.0`, but that did not change anything.

I then checked to make sure that the docker port was not being forwarded to another port

```bash
$ docker port mysql 3306
0.0.0.0:3306
```

#### Changing Docker Port

To try and eliminate the possibility that a local instance onf MySQL was conflicting on port 3306, I updated my `MYSQL_PORT` environment variable to `3307` in my [.env](https://github.com/Kelley12/horsin-around/blob/develop/.env.example) file.

```bash
# Remove local docker volume
$ rm -rf db_data/

# Remove docker images
$ docker-compose rm -v
Are you sure? [yN] y
Removing phpmyadmin ... done
Removing mysql      ... done

# Check docker container processes
$ docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                    NAMES
be4246d0d654        phpmyadmin/phpmyadmin   "/docker-entrypoint.…"   6 minutes ago       Up 6 minutes        0.0.0.0:8080->80/tcp     phpmyadmin
5a38be2c5365        mysql:5.7.21            "docker-entrypoint.s…"   6 minutes ago       Up 6 minutes        0.0.0.0:3307->3306/tcp   mysql

# Check port mapping
$ docker port mysql 3306
0.0.0.0:3307

# Attempt connection
$ mysql --host=localhost --user=admin --password=admin --database=horsin-around --port=3307
ERROR 1045 (28000): Access denied for user 'admin'@'localhost' (using password: YES)
```

I was again able to successfully connect using MySQL Workbench using the new port.

#### Explicit Permissions

There were so many StockOverflow answers that resulted in wrong password or user didn't have the correct privileges. Because of this, I went into `PHPMyAdmin` and explicitely created and entry for `admin@localhost`

![Explicit Admin Privileges](/assets/images/phpmyadmin-admin-user-privileges.png "Explicit Admin Privileges")

But trying to connect led to the same old errors

```bash
# TypeORM Connection
Error - ER_ACCESS_DENIED_ERROR: Access denied for user 'admin'@'localhost' (using password: YES)

# Command line client
ERROR 1045 (28000): Access denied for user 'admin'@'localhost' (using password: YES)
```

## The Solution

I am extremely disapointed to not know what exactly fixed it, I tried again and it just worked. If I had to guess it was the [Changing Docker Port](#changing-docker-port), I think what might have happened is that the .env hadn't been pulled in when it was run. I unfortunately do not know. But hopefully this post gives some ideas for things to try and checks to make.
