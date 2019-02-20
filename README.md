# What is Apache Airflow Scheduler?

> Airflow is a platform to programmatically author, schedule and monitor workflows. Airflow Scheduler is one of the required components when the CeleryExecutor is configured.

https://airflow.apache.org/

# TL;DR;

## Docker Compose

```bash
$ curl -LO https://raw.githubusercontent.com/bitnami/bitnami-docker-airflow-scheduler/master/docker-compose.yml
$ docker-compose up
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.
* Bitnami container images are released daily with the latest distribution packages available.


> This [CVE scan report](https://quay.io/repository/bitnami/airflow-scheduler?tab=tags) contains a security report with all open CVEs. To get the list of actionable security issues, find the "latest" tag, click the vulnerability report link under the corresponding "Security scan" field and then select the "Only show fixable" filter on the next page.


# Supported tags and respective `Dockerfile` links

Learn more about the Bitnami tagging policy and the difference between rolling tags and immutable tags [in our documentation page](https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/).


* [`1-rhel-7`, `1.10.2-rhel-7-r0` (1/rhel-7/Dockerfile)](https://github.com/bitnami/bitnami-docker-airflow-scheduler/blob/1.10.2-rhel-7-r0/1/rhel-7/Dockerfile)
* [`1-ol-7`, `1.10.2-ol-7-r11` (1/ol-7/Dockerfile)](https://github.com/bitnami/bitnami-docker-airflow-scheduler/blob/1.10.2-ol-7-r11/1/ol-7/Dockerfile)
* [`1-debian-9`, `1.10.2-debian-9-r15`, `1`, `1.10.2`, `1.10.2-r15`, `latest` (1/debian-9/Dockerfile)](https://github.com/bitnami/bitnami-docker-airflow-scheduler/blob/1.10.2-debian-9-r15/1/debian-9/Dockerfile)

Subscribe to project updates by watching the [bitnami/airflow GitHub repo](https://github.com/bitnami/bitnami-docker-airflow-scheduler).

# Prerequisites

To run this application you need [Docker Engine](https://www.docker.com/products/docker-engine) >= `1.10.0`. [Docker Compose](https://www.docker.com/products/docker-compose) is recommended with a version `1.6.0` or later.

# How to use this image

Airflow Scheduler is a component of an Airflow solution configuring with the `CeleryExecutor`. Hence, you will need to rest of Airflow components for this image to work.
You will need an [Airflow Webserver](https://www.github.com/bitnami/bitnami-docker-airflow), one or more [Airflow Workers](https://www.github.com/bitnami/bitnami-docker-airflow-worker), a [PostgreSQL database](https://www.github.com/bitnami/bitnami-docker-postgresql) and a [Redis server](https://www.github.com/bitnami/bitnami-docker-redis).

## Using Docker Compose

The recommended way to run Airflow Scheduler is using Docker Compose using the following `docker-compose.yml` template:

```yaml
version: '2'
services:
  postgresql:
    image: 'bitnami/postgresql:latest'
    volumes:
      - 'postgresql_data:/bitnami'
    environment:
      - POSTGRESQL_DATABASE=bitnami_airflow
      - POSTGRESQL_USERNAME=bn_airflow
      - POSTGRESQL_PASSWORD=bitnami1
  redis:
    image: 'bitnami/redis:latest'
    volumes:
      - 'redis_data:/bitnami'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
  airflow-worker:
    image: bitnami/airflow-worker:latest
    environment:
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USER=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
    volumes:
      - airflow_worker_data:/bitnami
  airflow-scheduler:
    image: bitnami/airflow-scheduler:latest
    environment:
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USER=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_LOAD_EXAMPLES=yes
    volumes:
      - airflow_scheduler_data:/bitnami
  airflow:
    image: bitnami/airflow:latest
    environment:
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USER=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_PASSWORD=bitnami123
      - AIRFLOW_USERNAME=user
      - AIRFLOW_EMAIL=user@example.com
    ports:
      - '8080:8080'
    volumes:
      - airflow_data:/bitnami
volumes:
  airflow_scheduler_data:
    driver: local
  airflow_worker_data:
    driver: local
  airflow_data:
    driver: local
  postgresql_data:
    driver: local
  redis_data:
    driver: local
```

Launch the containers using:

```bash
$ docker-compose up -d
```

## Using the Docker Command Line

If you want to run the application manually instead of using `docker-compose`, these are the basic steps you need to run:

1. Create a network

  ```bash
  $ docker network create airflow-tier
  ```

2. Create a volume for PostgreSQL persistence and create a PostgreSQL container

  ```bash
  $ docker volume create --name postgresql_data
  $ docker run -d --name postgresql \
    -e POSTGRESQL_USERNAME=bn_airflow \
    -e POSTGRESQL_PASSWORD=bitnami1 \
    -e POSTGRESQL_DATABASE=bitnami_airflow \
    --net airflow-tier \
    --volume postgresql_data:/bitnami \
    bitnami/postgresql:latest
  ```

3. Create a volume for Redis persistence and create a Redis container

  ```bash
  $ docker volume create --name redis_data
  $ docker run -d --name redis \
    -e ALLOW_EMPTY_PASSWORD=yes \
    --net airflow-tier \
    --volume redis_data:/bitnami \
    bitnami/redis:latest
  ```

4. Create volumes for Airflow persistence and launch the container

  ```bash
  $ docker volume create --name airflow_data
  $ docker run -d --name airflow -p 8080:8080 \
    -e AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho= \
    -e AIRFLOW_EXECUTOR=CeleryExecutor \
    -e AIRFLOW_DATABASE_NAME=bitnami_airflow \
    -e AIRFLOW_DATABASE_USER=bn_airflow \
    -e AIRFLOW_DATABASE_PASSWORD=bitnami1 \
    -e AIRFLOW_PASSWORD=bitnami123 \
    -e AIRFLOW_USERNAME=user \
    -e AIRFLOW_EMAIL=user@example.com \
    --net airflow-tier \
    --volume airflow_data:/bitnami \
    bitnami/airflow:latest
  ```

5. Create volumes for Airflow Scheduler persistence and launch the container

  ```bash
  $ docker volume create --name airflow_scheduler_data
  $ docker run -d --name airflow-scheduler \
    -e AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho= \
    -e AIRFLOW_EXECUTOR=CeleryExecutor \
    -e AIRFLOW_DATABASE_NAME=bitnami_airflow \
    -e AIRFLOW_DATABASE_USER=bn_airflow \
    -e AIRFLOW_DATABASE_PASSWORD=bitnami1 \
    -e AIRFLOW_LOAD_EXAMPLES=yes \
    --net airflow-tier \
    --volume airflow_scheduler_data:/bitnami \
    bitnami/airflow-scheduler:latest
  ```

6. Create volumes for Airflow Worker persistence and launch the container

  ```bash
  $ docker volume create --name airflow_worker_data
  $ docker run -d --name airflow-worker \
    -e AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho= \
    -e AIRFLOW_EXECUTOR=CeleryExecutor \
    -e AIRFLOW_DATABASE_NAME=bitnami_airflow \
    -e AIRFLOW_DATABASE_USER=bn_airflow \
    -e AIRFLOW_DATABASE_PASSWORD=bitnami1 \
    --net airflow-tier \
    --volume airflow_worker_data:/bitnami \
    bitnami/airflow-worker:latest
  ```

Access your application at http://your-ip:8080

## Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount volumes for persistence of [PostgreSQL data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database) and [Redis data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database)

The above examples define docker volumes namely `postgresql_data`, `redis_data`, `airflow_data`, `airflow_scheduler_data` and `airflow_worker_data`. The Airflow Scheduler application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

The following `docker-compose.yml` template demonstrates the use of host directories as data volumes.

```yaml
version: '2'
services:
  postgresql:
    image: 'bitnami/postgresql:latest'
    environment:
      - POSTGRESQL_DATABASE=bitnami_airflow
      - POSTGRESQL_USERNAME=bn_airflow
      - POSTGRESQL_PASSWORD=bitnami1
    volumes:
      - /path/to/airflow-persistence:/bitnami
  redis:
    image: 'bitnami/redis:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - /path/to/airflow-persistence:/bitnami
  airflow-worker:
    image: bitnami/airflow-worker:latest
    environment:
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USER=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
    volumes:
      - /path/to/airflow-persistence:/bitnami
  airflow-scheduler:
    image: bitnami/airflow-scheduler:latest
    environment:
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USER=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_LOAD_EXAMPLES=yes
    volumes:
      - /path/to/airflow-persistence:/bitnami
  airflow:
    image: bitnami/airflow:latest
    environment:
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USER=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_PASSWORD=bitnami123
      - AIRFLOW_USERNAME=user
      - AIRFLOW_EMAIL=user@example.com
    ports:
      - '8080:8080'
    volumes:
      - /path/to/airflow-persistence:/bitnami
```

### Mount host directories as data volumes using the Docker command line

1. Create a network (if it does not exist)

  ```bash
  $ docker network create airflow-tier
  ```

2. Create a volume for PostgreSQL persistence and create a PostgreSQL container

  ```bash
  $ docker volume create --name postgresql_data
  $ docker run -d --name postgresql \
    -e POSTGRESQL_USERNAME=bn_airflow \
    -e POSTGRESQL_PASSWORD=bitnami1 \
    -e POSTGRESQL_DATABASE=bitnami_airflow \
    --net airflow-tier \
    --volume /path/to/postgresql-persistence:/bitnami \
    bitnami/postgresql:latest
  ```

3. Create a volume for Redis persistence and create a Redis container

  ```bash
  $ docker volume create --name redis_data
  $ docker run -d --name redis \
    -e ALLOW_EMPTY_PASSWORD=yes \
    --net airflow-tier \
    --volume /path/to/redis-persistence:/bitnami \
    bitnami/redis:latest
  ```

4. Create volumes for Airflow persistence and launch the container

  ```bash
  $ docker volume create --name airflow_data
  $ docker run -d --name airflow -p 8080:8080 \
    -e AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho= \
    -e AIRFLOW_EXECUTOR=CeleryExecutor \
    -e AIRFLOW_DATABASE_NAME=bitnami_airflow \
    -e AIRFLOW_DATABASE_USER=bn_airflow \
    -e AIRFLOW_DATABASE_PASSWORD=bitnami1 \
    -e AIRFLOW_PASSWORD=bitnami123 \
    -e AIRFLOW_USERNAME=user \
    -e AIRFLOW_EMAIL=user@example.com \
    --net airflow-tier \
    --volume /path/to/airflow-persistence:/bitnami \
    bitnami/airflow:latest
  ```

5. Create volumes for Airflow Scheduler persistence and launch the container

  ```bash
  $ docker volume create --name airflow_scheduler_data
  $ docker run -d --name airflow-scheduler \
    -e AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho= \
    -e AIRFLOW_EXECUTOR=CeleryExecutor \
    -e AIRFLOW_DATABASE_NAME=bitnami_airflow \
    -e AIRFLOW_DATABASE_USER=bn_airflow \
    -e AIRFLOW_DATABASE_PASSWORD=bitnami1 \
    -e AIRFLOW_LOAD_EXAMPLES=yes \
    --net airflow-tier \
    --volume /path/to/airflow-scheduler-persistence:/bitnami \
    bitnami/airflow-scheduler:latest
  ```

6. Create volumes for Airflow Worker persistence and launch the container

  ```bash
  $ docker volume create --name airflow_worker_data
  $ docker run -d --name airflow-worker \
    -e AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho= \
    -e AIRFLOW_EXECUTOR=CeleryExecutor \
    -e AIRFLOW_DATABASE_NAME=bitnami_airflow \
    -e AIRFLOW_DATABASE_USER=bn_airflow \
    -e AIRFLOW_DATABASE_PASSWORD=bitnami1 \
    --net airflow-tier \
    --volume /path/to/airflow-worker-persistence:/bitnami \
    bitnami/airflow-worker:latest
  ```

# Configuration

## Environment variables

The Airflow Scheduler instance can be customized by specifying environment variables on the first run. The following environment values are provided to customize Airflow Scheduler:

##### Airflow Scheduler configuration

- `AIRFLOW_EXECUTOR`: Airflow Scheduler executor. Default: **SequentialExecutor**
- `AIRFLOW_FERNET_KEY`: Airflow Scheduler Fernet key. No defaults.
- `AIRFLOW_WEBSERVER_HOST`: Airflow Scheduler webserver host. Default: **127.0.0.1**
- `AIRFLOW_WEBSERVER_PORT_NUMBER`: Airflow Scheduler webserver port. Default: **8080**
- `AIRFLOW_LOAD_EXAMPLES`: To load example tasks into the application. Default: **yes**
- `AIRFLOW_HOSTNAME_CALLABLE`: Method to obtain the hostname. No defaults.

##### Use an existing database

- `AIRFLOW_DATABASE_HOST`: Hostname for PostgreSQL server. Default: **postgresql**
- `AIRFLOW_DATABASE_PORT_NUMBER`: Port used by PostgreSQL server. Default: **5432**
- `AIRFLOW_DATABASE_NAME`: Database name that Airflow Scheduler will use to connect with the database. Default: **bitnami_airflow**
- `AIRFLOW_DATABASE_USERNAME`: Database user that Airflow Scheduler will use to connect with the database. Default: **bn_airflow**
- `AIRFLOW_DATABASE_PASSWORD`: Database password that Airflow Scheduler will use to connect with the database. No defaults.
- `AIRFLOW_DATABASE_USE_SSL`: Set to yes if the database uses SSL. Default: **no**
- `AIRFLOW_REDIS_USE_SSL`: Set to yes if Redis uses SSL. Default: **no**
- `REDIS_HOST`: Hostname for Redis server. Default: **redis**
- `REDIS_PORT_NUMBER`: Port used by Redis server. Default: **6379**
- `REDIS_USER`: USER that Airflow Scheduler will use to connect with Redis. No defaults.
- `REDIS_PASSWORD`: Password that Airflow Scheduler will use to connect with Redis. No defaults.

> In addition to the previous environment variables, all the parameters from the configuration file can be overwritten by using environment variables with this format: `AIRFLOW__{SECTION}__{KEY}`. Note the double underscores.

### Specifying Environment variables using Docker Compose

```yaml
version: '2'

services:
  airflow:
    image: bitnami/airflow:1
    environment:
      - AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
      - AIRFLOW_EXECUTOR=CeleryExecutor
      - AIRFLOW_DATABASE_NAME=bitnami_airflow
      - AIRFLOW_DATABASE_USER=bn_airflow
      - AIRFLOW_DATABASE_PASSWORD=bitnami1
      - AIRFLOW_PASSWORD=bitnami123
      - AIRFLOW_USERNAME=user
      - AIRFLOW_EMAIL=user@example.com
```

### Specifying Environment variables on the Docker command line

```bash
$ docker run -d --name airflow -p 8080:8080 \
    -e AIRFLOW_FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho= \
    -e AIRFLOW_EXECUTOR=CeleryExecutor \
    -e AIRFLOW_DATABASE_NAME=bitnami_airflow \
    -e AIRFLOW_DATABASE_USER=bn_airflow \
    -e AIRFLOW_DATABASE_PASSWORD=bitnami1 \
    -e AIRFLOW_PASSWORD=bitnami123 \
    -e AIRFLOW_USERNAME=user \
    -e AIRFLOW_EMAIL=user@example.com \
    --volume airflow_data:/bitnami \
    bitnami/airflow:latest
```

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-airflow-scheduler/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-airflow-scheduler/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-airflow-scheduler/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`$ docker version`)
- Output of `$ docker info`
- Version of this container (`$ echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# License

Copyright 2015-2019 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.