# Sonar On Docker

A Docker image contains **SonarQube** + **MySQL**

[![](https://img.shields.io/badge/Docker%20Hub-info-blue.svg)](https://hub.docker.com/r/thyrlian/sonar/)
[![Build Status](https://travis-ci.org/thyrlian/SonarOnDocker.svg?branch=master)](https://travis-ci.org/thyrlian/SonarOnDocker)

<img src="https://github.com/thyrlian/SonarOnDocker/blob/master/Banner.png">

```
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
```

## Persist Data

* ```/var/lib/mysql```

All historical analysis data, imported rules, changed settings are saved here.

* ```/opt/sonarqube/extensions```

SonarQube's plugins.

* ```/opt/sonarqube/data/es```

**Optional**: ElasticSearch indices, no need to be persisted, will be auto-generated.

## Upgrading

### MySQL

1. Perform a logical backup on the old version of MySQL

    ```bash
    mysqldump -u sonar -p --opt sonar > [path_to_mysql_backup]/sonar.sql
    ```

2. Start a MySQL docker container (new version of MySQL)

    ```bash
    docker run -i -t -v [path_to_mysql_backup]:/tmp -v [path_to_persist_db]:/var/lib/mysql mysql /bin/bash
    ```

3. Start MySQL server

    ```bash
    /etc/init.d/mysql start
    ```

4. Start MySQL client

    ```bash
    mysql
    ```

5. Create and use the database

    ```sql
    create database sonar;
    use sonar;
    ```

6. Grant privileges to user

    ```sql
    grant all on sonar.* to 'sonar'@'%' identified by 'sonar';
    grant all on sonar.* to 'sonar'@'localhost' identified by 'sonar';
    grant usage on *.* to sonar@localhost identified by 'sonar';
    grant all privileges on sonar.* to sonar@localhost;
    ```

7. Restore the backup file (by executing SQL script)

    ```sql
    source /tmp/sonar.sql
    ```

8. Quit MySQL client

    ```sql
    exit
    ```

9. Stop MySQL server

    ```bash
    /etc/init.d/mysql stop
    ```

Now you have successfully restored the database on the new version of MySQL.  The database data are stored in ***path_to_persist_db*** of your host.

### SonarQube

## N.B.

There is a permission problem when mounting a host directory in MySQL container using boot2docker.

```console
[ERROR] InnoDB: Operating system error number 13 in a file operation.
[ERROR] InnoDB: The error means mysqld does not have the access rights to the directory.
```

So if you use Mac OS X, please try the approach below:

1. Build a custom MySQL image:

    ```console
    docker build -t mysql_mac mysql_mac/
    ```
2. Edit *docker-compose.yml*, replace `image: mysql` by `image: mysql_mac`.
