# Traefik Dockerized

- [Traefik Dockerized](#traefik-dockerized)
  - [Overview](#overview)
  - [Requirements](#requirements)
  - [Installation](#installation)
    - [Traefik Dashboard](#traefik-dashboard)
    - [Entry Points](#entry-points)
    - [Docker Network](#docker-network)
    - [Let's Encrypt](#lets-encrypt)
    - [acme.json](#acmejson)
    - [Dashboard Routing](#dashboard-routing)
  - [Usage](#usage)
    - [Update Traefik](#update-traefik)
  - [Resources](#resources)

---

## Overview

This repository contains a template to deploy [Traefik 2](https://containo.us/traefik/) using [Docker Compose](https://docs.docker.com/compose/) on a single machine running [Docker](https://www.docker.com/).

## Requirements

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Git](https://git-scm.com/)
- Text editor of your choice (e.g. [Vim](https://www.vim.org/))

## Installation

Clone the repository:

```sh
$ git clone https://github.com/cedrichopf/traefik-dockerized.git
Cloning into 'traefik-dockerized'...
```

Create a copy of the example configuration files:

```sh
# Traefik Configuration
$ cp config/traefik.example.yml config/traefik.yml
# Custom Docker Compose Configuration
$ cp override.example.yml docker-compose.override.yml
```

### Traefik Dashboard

To disable the _Traefik_ Dashboard, change the following configuration value to `false`:

```yaml
api:
  dashboard: false
```

### Entry Points

Per default, this Traefik deployment listens on port `80` (HTTP) and `443` (HTTPS). This can be changed by adapting the `address` field of the Entry Points:

```yaml
entryPoints:
  http:
    address: ":80"
  https:
    address: ":443"
```

### Docker Network

To let _Traefik_ auto-discover the applications running as a Docker container on the machine, create a Docker network and add it to the configuration. In this example, the Docker network is called `proxy`.

1. Create a Docker network:

```sh
$ docker network create proxy
ca0a9fe39b34b9f17d5c5e938e82ce67b4423e151ae5000eee7754e89116cac1
```

1. Add the network to the configuration:

```yaml
providers:
  docker:
    network: proxy
```

### Let's Encrypt

To use the built-in Let's Encrypt support, add a Certificate Resolver to the configuration:

```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@example.com
      storage: acme.json
      httpChallenge:
        entryPoint: http
```

### acme.json

The file `acme.json` will be mounted inside the _Traefik_ container and is used to store the certificates received from Let's Encrypt. Create this file and change the file permissions to `600`:

```sh
$ touch letsencrypt/acme.json
$ chmod 600 letsencrypt/acme.json
```

### Dashboard Routing

If the _Traefik_ Dashboard is enabled, configure the router in the `docker-compose.override.yml` file to make the dashboard available:

```yaml
labels:
  - traefik.http.routers.traefik-http.rule=Host(`traefik.example.com`)
  - traefik.http.routers.traefik-http.entrypoints=http
  - traefik.http.routers.traefik-http.middlewares=redirect
  - traefik.http.routers.traefik-https.rule=Host(`traefik.example.com`)
  - traefik.http.routers.traefik-https.entrypoints=https
  - traefik.http.routers.traefik-https.tls=true
  - traefik.http.routers.traefik-http.service=api@internal
  - traefik.http.routers.traefik-https.service=api@internal
  - traefik.http.middlewares.redirect.redirectscheme.scheme=https
```

## Usage

Once the configuration is completed, download the Docker images and start the services using docker-compose:

```sh
$ docker-compose pull
$ docker-compose up -d
```

To stop the deployment, you can either run the `stop` or `down` command of docker-compose:

```sh
$ docker-compose stop
Stopping traefik_traefik_1 ... done
$ docker-compose down
Stopping traefik_traefik_1 ... done
Removing traefik_traefik_1 ... done
```

By using `docker-compose down` instead of `docker-compose stop`, the containers will be also removed.

### Update Traefik

To update the _Traefik_ instance, download the latest Docker images and recreate the services:

```sh
$ docker-compose pull
Pulling traefik ... done
$ docker-compose up -d
Recreating traefik_traefik_1 ... done
```

The command `docker-compose pull` will automatically fetch and download the latest version of _Traefik_ available on Docker Hub. Finally, the command `docker-compose up -d` will recreate the running _Traefik_ container with the latest version.

## Resources

- [Traefik](https://containo.us/traefik/)
- [Traefik Documentation](https://docs.traefik.io/)
- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
