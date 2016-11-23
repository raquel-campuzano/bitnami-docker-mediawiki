[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-mediawiki/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-mediawiki/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/mediawiki)](https://hub.docker.com/r/bitnami/mediawiki/)

# What is Mediawiki?

> MediaWiki is an extremely powerful, scalable software and a feature-rich wiki implementation that uses PHP to process and display data stored in a database, such as MySQL.

Pages use MediaWiki's wikitext format, so that users without knowledge of XHTML or CSS can edit them easily.

https://www.mediawiki.org/

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

# How to use this image

## Run Mediawiki with a Database Container

Running Mediawiki with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run Mediawiki. You can use the following docker compose template:

```yaml
version: '2'
services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
  application:
    image: 'bitnami/mediawiki:latest'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - 'mediawiki_data:/bitnami/mediawiki'
      - 'apache_data:/bitnami/apache'
    depends_on:
      - mariadb
volumes:
  mariadb_data:
    driver: local
  mediawiki_data:
    driver: local
  apache_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create mediawiki-tier
  ```

2. Start a MariaDB database in the network generated:

  ```bash
  $ docker run -d --name mariadb --net=mediawiki-tier bitnami/mariadb
  ```

  *Note:* You need to give the container a name in order to Mediawiki to resolve the host

3. Run the Mediawiki container:

  ```bash
  $ docker run -d -p 80:80 --name mediawiki --net=mediawiki-tier bitnami/mediawiki
  ```

Then you can access your application at http://your-ip/

## Persisting your application


If you remove every container and volume all your data will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence of the MediaWiki deployment, the above examples define docker volumes namely `mariadb_data` and `mediawiki_data`. The MediaWiki application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

> **Note!** If you have already started using your application, follow the steps on [backing](#backing-up-your-application) up to pull the data from your running container down to your host.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:
```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    volumes:
      - /path/to/mariadb-persistence:/bitnami/mariadb
  mediawiki:
    image: 'bitnami/mediawiki:latest'
    depends_on:
      - mariadb
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '/path/to/mediawiki-persistence:/bitnami/mediawiki'
      - '/path/to/apache-persistence:/bitnami/apache'
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create mediawiki-tier
  ```

2. Create a MariaDB container with host volume:

  ```bash
  $ docker run -d --name mariadb \
    --net mediawiki-tier \
    --volume /path/to/mariadb-persistence:/bitnami/mariadb \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order to Mediawiki to resolve the host

3. Run the Mediawiki container:

  ```bash
  $ docker run -d --name mediawiki -p 80:80 -p 443:443 \
    --net mediawiki-tier \
    --volume /path/to/mediawiki-persistence:/bitnami/mediawiki \
    --volume /path/to/apache-persistence:/bitnami/apache \
    bitnami/mediawiki:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and Mediawiki, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Mediawiki container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/mediawiki:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop mediawiki`
 * For manual execution: `$ docker stop mediawiki`

3. (For non-compose execution only) Create a [backup](#backing-up-your-application) if you have not mounted the mediawiki folder in the host.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v mediawiki`
 * For manual execution: `$ docker rm -v mediawiki`

5. Run the new image

 * For docker-compose: `$ docker-compose start mediawiki`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name mediawiki bitnami/mediawiki:latest`

# Configuration
## Environment variables
 When you start the mediawiki image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:
```yaml
application:
  image: bitnami/mediawiki:latest
  ports:
    - 80:80
  environment:
    - MEDIAWIKI_PASSWORD=my_password
  volumes_from:
    - application_data
```

 * For manual execution add a `-e` option with each variable and value:

```bash
 $ docker run -d -e MEDIAWIKI_PASSWORD=my_password -p 80:80 --name mediawiki -v /your/local/path/bitnami/mediawiki:/bitnami/mediawiki --network=mediawiki-tier bitnami/mediawiki
```

Available variables:

 - `MEDIAWIKI_USERNAME`: Mediawiki application username. Default: **user**
 - `MEDIAWIKI_PASSWORD`: Mediawiki application password. Default: **bitnami1**
 - `MEDIAWIKI_EMAIL`: Mediawiki application email. Default: **user@example.com**
 - `MEDIAWIKI_WIKI_NAME`: Mediawiki wiki name. Default: **Bitnami MediaWiki**
 - `MARIADB_USER`: Root user for the MariaDB database. Default: **root**
 - `MARIADB_PASSWORD`: Root password for the MariaDB.
 - `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
 - `MARIADB_PORT`: Port used by MariaDB server. Default: **3306**

### SMTP Configuration

To configure Mediawiki to send email using SMTP you can set the following environment variables:
 - `SMTP_HOST`: SMTP host.
 - `SMTP_HOST_ID`: SMTP host ID.
 - `SMTP_PORT`: SMTP port.
 - `SMTP_USER`: SMTP account user.
 - `SMTP_PASSWORD`: SMTP account password.

This would be an example of SMTP configuration using a GMail account:

 * docker-compose:

```yaml
  application:
    image: bitnami/mediawiki:latest
    ports:
      - 80:80
    environment:
      - SMTP_HOST=ssl://smtp.gmail.com
      - SMTP_HOST_ID=mydomain.com
      - SMTP_PORT=465
      - SMTP_USER=your_email@gmail.com
      - SMTP_PASSWORD=your_password
```
 * For manual execution:

```bash
 $ docker run -d -e SMTP_HOST=ssl://smtp.gmail.com -e SMTP_HOST_ID=mydomain.com -e SMTP_PORT=465 -e SMTP_USER=your_email@gmail.com -e SMTP_PASSWORD=your_password -p 80:80 --name mediawiki --net=mediawiki-tier bitnami/mediawiki
```

# Backing up your application

To backup your application data follow these steps:

1. Stop the running container:

  * For docker-compose: `$ docker-compose stop mediawiki`
  * For manual execution: `$ docker stop mediawiki`

2. Copy the Mediawiki data folder in the host:

  ```bash
  $ docker cp /your/local/path/bitnami:/bitnami/mediawiki
  ```

# Restoring a backup

To restore your application using backed up data simply mount the folder with Mediawiki data in the container. See [persisting your application](#persisting-your-application) section for more info.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an
[issue](https://github.com/bitnami/bitnami-docker-mediawiki/issues), or submit a
[pull request](https://github.com/bitnami/bitnami-docker-mediawiki/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an
[issue](https://github.com/bitnami/bitnami-docker-mediawiki/issues). For us to provide better support,
be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive
information)

# License

Copyright 2016 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
