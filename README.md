[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-mariadb/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-mariadb/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/mariadb)](https://hub.docker.com/r/bitnami/mariadb/)

# What is MariaDB?

> MariaDB is a fast, reliable, scalable, and easy to use open-source relational database system. MariaDB Server is intended for mission-critical, heavy-load production systems as well as for embedding into mass-deployed software.

[https://mariadb.com/](https://mariadb.com/)

# TLDR

```bash
docker run --name mariadb -e ALLOW_EMPTY_PASSWORD=yes bitnami/mariadb:latest
```

## Docker Compose

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - '3306:3306'
```

# Get this image

The recommended way to get the Bitnami MariaDB Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/mariadb).

```bash
docker pull bitnami/mariadb:latest
```

To use a specific version, you can pull a versioned tag. You can view the
[list of available versions](https://hub.docker.com/r/bitnami/mariadb/tags/)
in the Docker Hub Registry.

```bash
docker pull bitnami/mariadb:[TAG]
```

If you wish, you can also build the image yourself.

```bash
docker build -t bitnami/mariadb:latest https://github.com/bitnami/bitnami-docker-mariadb.git
```

# Persisting your database

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

**Note!**
If you have already started using your database, follow the steps on
[backing up](#backing-up-your-container) and [restoring](#restoring-a-backup) to pull the data from your running container down to your host.

The image exposes a volume at `/bitnami/mariadb` for the MariaDB data and configurations. For persistence you can mount a directory at this location from your host. If the mounted directory is empty, it will be initialized on the first run.

```bash
docker run -v /path/to/mariadb-persistence:/bitnami/mariadb bitnami/mariadb:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - '3306:3306'
    volumes:
      - /path/to/mariadb-persistence:/bitnami/mariadb
```

# Connecting to other containers

Using [Docker container networking](https://docs.docker.com/engine/userguide/networking/), a MariaDB server running inside a container can easily be accessed by your application containers.

Containers attached to the same network can communicate with each other using the container name as the hostname.

## Using the Command Line

In this example, we will create a MariaDB client instance that will connect to the server instance that is running on the same docker network as the client.

### Step 1: Create a network

```bash
$ docker network create app-tier --driver bridge
```

### Step 2: Launch the MariaDB server instance

Use the `--network app-tier` argument to the `docker run` command to attach the MariaDB container to the `app-tier` network.

```bash
$ docker run -d --name mariadb-server \
    -e ALLOW_EMPTY_PASSWORD=yes \
    --network app-tier \
    bitnami/mariadb:latest
```

### Step 3: Launch your MariaDB client instance

Finally we create a new container instance to launch the MariaDB client and connect to the server created in the previous step:

```bash
$ docker run -it --rm \
    --network app-tier \
    bitnami/mariadb:latest mysql -h mariadb-server -u root
```

## Using Docker Compose

When not specified, Docker Compose automatically sets up a new network and attaches all deployed services to that network. However, we will explicitly define a new `bridge` network named `app-tier`. In this example we assume that you want to connect to the MariaDB server from your own custom application image which is identified in the following snippet by the service name `myapp`.

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    networks:
      - app-tier
  myapp:
    image: 'YOUR_APPLICATION_IMAGE'
    networks:
      - app-tier
```

> **IMPORTANT**:
>
> 1. Please update the `YOUR_APPLICATION_IMAGE` placeholder in the above snippet with your application image
> 2. In your application container, use the hostname `mariadb` to connect to the MariaDB server

Launch the containers using:

```bash
$ docker-compose up -d
```

# Configuration

## Setting the root password on first run

The root user and password can easily be setup with the Bitnami MariaDB Docker image using the following environment variables:

 - `MARIADB_ROOT_USER`: The database admin user. Defaults to `root`.
 - `MARIADB_ROOT_PASSWORD`: The database admin user password. No defaults.

Passing the `MARIADB_ROOT_PASSWORD` environment variable when running the image for the first time will set the password of the `MARIADB_ROOT_USER` user to the value of `MARIADB_ROOT_PASSWORD`.

```bash
docker run --name mariadb -e MARIADB_ROOT_PASSWORD=password123 bitnami/mariadb:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    ports:
      - '3306:3306'
    environment:
      - MARIADB_ROOT_PASSWORD=password123
```

**Warning** The `MARIADB_ROOT_USER` user is always created with remote access. It's suggested that the `MARIADB_ROOT_PASSWORD` env variable is always specified to set a password for the `MARIADB_ROOT_USER` user. In case you want to allow the `MARIADB_ROOT_USER` user to access the database without a password set the environment variable `ALLOW_EMPTY_PASSWORD=yes`. **This is recommended only for development**.

## Allowing empty passwords

By default the MariaDB image expects that all the available password to be set. In order to allow empty passwords, it is necessary to set the `ALLOW_EMPTY_PASSWORD` env variable. This env variable is only recommended for testing or development purpose. We strongly recommend to specify the `MARIADB_ROOT_PASSWORD` for any other scenario.

```bash
docker run --name mariadb -e ALLOW_EMPTY_PASSWORD=yes bitnami/mariadb:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    ports:
      - '3306:3306'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
```

## Creating a database on first run

By passing the `MARIADB_DATABASE` environment variable when running the image for the first time, a database will be created. This is useful if your application requires that a database already exists, saving you from having to manually create the database using the MySQL client.

```bash
docker run --name mariadb -e MARIADB_DATABASE=my_database bitnami/mariadb:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    ports:
      - '3306:3306'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_DATABASE=my_database
```

## Creating a database user on first run

You can create a restricted database user that only has permissions for the database created with the [`MARIADB_DATABASE`](#creating-a-database-on-first-run) environment variable. To do this, provide the `MARIADB_USER` environment variable and to set a password for the database user provide the `MARIADB_PASSWORD` variable.

```bash
docker run --name mariadb \
  -e ALLOW_EMPTY_PASSWORD=yes \
  -e MARIADB_USER=my_user \
  -e MARIADB_PASSWORD=my_password \
  -e MARIADB_DATABASE=my_database \
  bitnami/mariadb:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    ports:
      - '3306:3306'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=my_user
      - MARIADB_PASSWORD=my_password
      - MARIADB_DATABASE=my_database
```

**Note!** The `root` user will still be created with remote access if you set the `ALLOW_EMPTY_PASSWORD` env variable. Please provide the `MARIADB_ROOT_PASSWORD` env variable instead if you want to set a password for the `root` user.

## Setting up a replication cluster

A **zero downtime** MariaDB master-slave [replication](https://dev.mysql.com/doc/refman/5.0/en/replication-howto.html) cluster can easily be setup with the Bitnami MariaDB Docker image using the following environment variables:

 - `MARIADB_REPLICATION_MODE`: The replication mode. Possible values `master`/`slave`. No defaults.
 - `MARIADB_REPLICATION_USER`: The replication user created on the master on first run. No defaults.
 - `MARIADB_REPLICATION_PASSWORD`: The replication users password. No defaults.
 - `MARIADB_MASTER_HOST`: Hostname/IP of replication master (slave parameter). No defaults.
 - `MARIADB_MASTER_PORT`: Server port of the replication master (slave parameter). Defaults to `3306`.
 - `MARIADB_MASTER_ROOT_USER`: User on replication master with access to `MARIADB_DATABASE` (slave parameter). Defaults to `root`
 - `MARIADB_MASTER_ROOT_PASSWORD`: Password of user on replication master with access to `MARIADB_DATABASE` (slave parameter). No defaults.

In a replication cluster you can have one master and zero or more slaves. When replication is enabled the master node is in read-write mode, while the slaves are in read-only mode. For best performance its advisable to limit the reads to the slaves.

### Step 1: Create the replication master

The first step is to start the MariaDB master.

```bash
docker run --name mariadb-master \
  -e MARIADB_ROOT_PASSWORD=root_password \
  -e MARIADB_REPLICATION_MODE=master \
  -e MARIADB_REPLICATION_USER=my_repl_user \
  -e MARIADB_REPLICATION_PASSWORD=my_repl_password \
  -e MARIADB_USER=my_user \
  -e MARIADB_PASSWORD=my_password \
  -e MARIADB_DATABASE=my_database \
  bitnami/mariadb:latest
```

In the above command the container is configured as the `master` using the `MARIADB_REPLICATION_MODE` parameter. A replication user is specified using the `MARIADB_REPLICATION_USER` and `MARIADB_REPLICATION_PASSWORD` parameters.

### Step 2: Create the replication slave

Next we start a MariaDB slave container.

```bash
docker run --name mariadb-slave --link mariadb-master:master \
  -e MARIADB_ROOT_PASSWORD=root_password \
  -e MARIADB_REPLICATION_MODE=slave \
  -e MARIADB_REPLICATION_USER=my_repl_user \
  -e MARIADB_REPLICATION_PASSWORD=my_repl_password \
  -e MARIADB_MASTER_HOST=master \
  -e MARIADB_MASTER_ROOT_USER=my_user \
  -e MARIADB_MASTER_ROOT_PASSWORD=my_password \
  -e MARIADB_USER=my_user \
  -e MARIADB_PASSWORD=my_password \
  -e MARIADB_DATABASE=my_database \
  bitnami/mariadb:latest
```

In the above command the container is configured as a `slave` using the `MARIADB_REPLICATION_MODE` parameter. The `MARIADB_MASTER_HOST`, `MARIADB_MASTER_ROOT_USER` and `MARIADB_MASTER_ROOT_PASSWORD` parameters are used by the slave to connect to the master and take a dump of the existing data in the database identified by `MARIADB_DATABASE`. The replication user credentials are specified using the `MARIADB_REPLICATION_USER` and `MARIADB_REPLICATION_PASSWORD` parameters and should be the same as the one specified on the master.

> **Note**! The cluster only replicates the database specified in the `MARIADB_DATABASE` parameter.

You now have a two node MariaDB master/slave replication cluster up and running. You can scale the cluster by adding/removing slaves without incurring any downtime.

With Docker Compose the master/slave replication can be setup using:

```yaml
version: '2'

services:
  mariadb-master:
    image: 'bitnami/mariadb:latest'
    ports:
      - '3306'
    volumes:
      - /path/to/mariadb-persistence:/bitnami/mariadb
    environment:
      - MARIADB_REPLICATION_MODE=master
      - MARIADB_REPLICATION_USER=repl_user
      - MARIADB_REPLICATION_PASSWORD=repl_password
      - MARIADB_ROOT_PASSWORD=root_password
      - MARIADB_USER=my_user
      - MARIADB_PASSWORD=my_password
      - MARIADB_DATABASE=my_database
  mariadb-slave:
    image: 'bitnami/mariadb:latest'
    ports:
      - '3306'
    depends_on:
      - mariadb-master
    environment:
      - MARIADB_REPLICATION_MODE=slave
      - MARIADB_REPLICATION_USER=repl_user
      - MARIADB_REPLICATION_PASSWORD=repl_password
      - MARIADB_MASTER_HOST=mariadb-master
      - MARIADB_MASTER_PORT=3306
      - MARIADB_MASTER_ROOT_USER=my_user
      - MARIADB_MASTER_ROOT_PASSWORD=my_password
      - MARIADB_ROOT_PASSWORD=root_password
      - MARIADB_USER=my_user
      - MARIADB_PASSWORD=my_password
      - MARIADB_DATABASE=my_database
```

Scale the number of slaves using:

```bash
docker-compose scale mariadb-master=1 mariadb-slave=3
```

The above command scales up the number of slaves to `3`. You can scale down in the same manner.

> **Note**: You should not scale up/down the number of master nodes. Always have only one master node running.

## Configuration file

The image looks for configuration in the `conf/` directory of `/bitnami/mariadb`. As mentioned in [Persisting your database](#persisting-your-data) you can mount a volume at this location and copy your own custom `my_custom.cnf` file in the `conf/` directory. That file will be included in the main configuration file and will overwrite any configuration you want to modify.

For example, in order to override the max_allowed_packet directive:

# Step 1: Write your my_custom.cnf file with the following content.
```
[mysqld]
max_allowed_packet=32M
```

# Step 2: Run the mariaDB image with the designed volume attached.
```
docker run --name mariadb -v /path/to/my_custom_cnf_directory:/bitnami/mariadb bitnami/mariadb:latest
```
After that, your changes will be taken into account in the server's behaviour.

### Step 1: Run the MariaDB image

Run the MariaDB image, mounting a directory from your host.

```bash
docker run --name mariadb -v /path/to/mariadb-persistence:/bitnami/mariadb bitnami/mariadb:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - '3306:3306'
    volumes:
      - /path/to/mariadb-persistence:/bitnami/mariadb
```

### Step 2: Edit the configuration

Edit the configuration on your host using your favorite editor.

```bash
vi /path/to/mariadb-persistence/conf/my.cnf
```

### Step 3: Restart MariaDB

After changing the configuration, restart your MariaDB container for changes to take effect.

```bash
docker restart mariadb
```

or using Docker Compose:

```bash
docker-compose restart mariadb
```

**Further Reading:**

  - [Server Option and Variable Reference](https://dev.mysql.com/doc/refman/5.1/en/mysqld-option-tables.html)

# Logging

The Bitnami MariaDB Docker image sends the container logs to the `stdout`. To view the logs:

```bash
docker logs mariadb
```

or using Docker Compose:

```bash
docker-compose logs mariadb
```

You can configure the containers [logging driver](https://docs.docker.com/engine/admin/logging/overview/) using the `--log-driver` option if you wish to consume the container logs differently. In the default configuration docker uses the `json-file` driver.

# Maintenance

## Backing up your container

To backup your data, configuration and logs, follow these simple steps:

### Step 1: Stop the currently running container

```bash
docker stop mariadb
```

or using Docker Compose:

```bash
docker-compose stop mariadb
```

### Step 2: Run the backup command

We need to mount two volumes in a container we will use to create the backup: a directory on your host to store the backup in, and the volumes from the container we just stopped so we can access the data.

```bash
docker run --rm -v /path/to/mariadb-backups:/backups --volumes-from mariadb busybox \
  cp -a /bitnami/mariadb:latest /backups/latest
```

or using Docker Compose:

```bash
docker run --rm -v /path/to/mariadb-backups:/backups --volumes-from `docker-compose ps -q mariadb` busybox \
  cp -a /bitnami/mariadb:latest /backups/latest
```

## Restoring a backup

Restoring a backup is as simple as mounting the backup as volumes in the container.

```bash
docker run -v /path/to/mariadb-backups/latest:/bitnami/mariadb bitnami/mariadb:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - '3306:3306'
    volumes:
      - /path/to/mariadb-backups/latest:/bitnami/mariadb
```

## Upgrade this image

Bitnami provides up-to-date versions of MariaDB, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container.

### Step 1: Get the updated image

```bash
docker pull bitnami/mariadb:latest
```

or if you're using Docker Compose, update the value of the image property to
`bitnami/mariadb:latest`.

### Step 2: Stop and backup the currently running container

Before continuing, you should backup your container's data, configuration and logs.

Follow the steps on [creating a backup](#backing-up-your-container).

### Step 3: Remove the currently running container

```bash
docker rm -v mariadb
```

or using Docker Compose:


```bash
docker-compose rm -v mariadb
```

### Step 4: Run the new image

Re-create your container from the new image, [restoring your backup](#restoring-a-backup) if necessary.

```bash
docker run --name mariadb bitnami/mariadb:latest
```

or using Docker Compose:

```bash
docker-compose start mariadb
```

# Notable Changes
## 10.1.21-r2

- `MARIADB_MASTER_USER` has been renamed to `MARIADB_MASTER_ROOT_USER`
- `MARIADB_MASTER_PASSWORD` has been renamed to `MARIADB_MASTER_ROOT_PASSWORD`
- `MARIADB_ROOT_USER` has been added to the available env variables. It can be used to specify the admin user.
- `ALLOW_EMPTY_PASSWORD` has been added to the available env variables. It can be used to allow blank passwords for MariaDB.
- By default the MariaDB image requires a root password to start. You can specify it using the `MARIADB_ROOT_PASSWORD` env variable or disable this requirement by setting the `ALLOW_EMPTY_PASSWORD`  env variable to `yes` (testing or development scenarios).

## 10.1.13-r0

- All volumes have been merged at `/bitnami/mariadb`. Now you only need to mount a single volume at `/bitnami/mariadb` for persistence.
- The logs are always sent to the `stdout` and are no longer collected in the volume.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-mariadb/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-mariadb/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-mariadb/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# License

Copyright (c) 2015-2016 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
