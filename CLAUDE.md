# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Docker-based infrastructure for running multiple PHP applications locally. It provides PHP-FPM 8.5, PostgreSQL 15, MongoDB, Nginx, and a Mailpit mail trap as isolated services in a custom bridge network. Application source code lives in a **sibling directory** (`../services/`) that is volume-mounted into both the PHP and Nginx containers.

## Setup

1. Copy `.env.example` to `.env` (adjust credentials if needed)
2. Start all services:
   ```sh
   docker-compose up --build
   ```

## Common Commands

```sh
# Start services (detached)
docker-compose up -d

# Rebuild PHP container after Dockerfile changes
docker-compose up --build php

# View logs
docker-compose logs -f php
docker-compose logs -f nginx

# Open a shell in the PHP container
docker-compose exec php bash
docker exec -it services_docker_php bash

# Run Composer inside the container
docker-compose exec php composer install --working-dir=/srv/src/<app-name>
```

## Architecture

```
docker-php-services/         ← this repo (infrastructure only)
../services/                 ← your PHP application code (mounted as /srv/src/)
```

**Network:** Custom bridge, subnet `172.20.0.0/16`

| Service    | Default container name      | IP            | Port mapping |
|------------|-----------------------------|---------------|--------------|
| PHP-FPM    | `services_docker_php`       | 172.20.0.10   | internal `9000` |
| PostgreSQL | `services_docker_postgres`  | 172.20.0.11   | `5432:5432` |
| MongoDB    | `services_docker_mongo`     | 172.20.0.12   | `27017:27017` |
| Nginx      | `services_docker_nginx`     | 172.20.0.30   | `8080:80` |
| Mailpit    | `services_docker_mailpit`   | 172.20.0.14   | `1025:1025`, `8025:8025` |

Container names come from `${NAME}` in `.env` (default `services_docker`); IPs come from the `*_IP` variables there.

**Request flow:** browser → host `:8080` → Nginx `:80` → FastCGI `php:9000` → PostgreSQL

Nginx config files placed in `./etc/nginx/` are mounted directly to `/etc/nginx/conf.d/`, so adding a new `.conf` file is picked up on container restart without a rebuild. Use `etc/nginx/example.conf` as the template for new virtual hosts — set `server_name`, `root`, and keep the `fastcgi_pass php:9000` directive.

## PHP Container

- Base: `php:8.5-fpm`, runs as non-root user `www` (uid 1001)
- PHP extensions: `pdo`, `pdo_pgsql`, `pgsql`, `pdo_mysql`, `gd`, `zip`, `exif`, `pcntl`, `soap`, `xml`, `mbstring`, `intl`, `mongodb 2.5.2`, XDebug
- Composer available globally
- OPcache is configured for local development in `etc/php/config/opcache.ini`

## XDebug

XDebug 3 is pre-installed and configured in `etc/php/config/xdebug.ini`:
- Mode: `develop,debug`
- Port: `9003`
- Client host: `host.docker.internal` (works with Docker Desktop; on Linux the `extra_hosts: host.docker.internal:host-gateway` entry in docker-compose.yml handles this)
- `PHP_IDE_CONFIG` is not set in `docker-compose.yml`; configure it in your IDE or container environment if your debugger setup needs a fixed server name

## Mail Trap (Mailpit)

All outgoing mail in local development goes to Mailpit instead of a real SMTP server:
- SMTP from inside the network: `mailpit:1025` (or `172.20.0.14:1025`)
- Web UI: `http://<host>:8025`
- HTTP API for automated tests: `GET http://<host>:8025/api/v1/messages`
- Messages persist in `./.data/mailpit/mailpit.db` across restarts; `MP_MAX_MESSAGES` (5000) caps the trap, dropping the oldest

Mailpit accepts everything and forwards nothing — never configure it as a transport in a production configuration.

## Adding a New Application

1. Place application code in `../services/<app-name>/`
2. Create `etc/nginx/<app-name>.conf` based on `etc/nginx/example.conf`, adjusting `server_name` and `root`
3. Restart Nginx: `docker-compose restart nginx`
4. Add the domain to your `/etc/hosts`: `127.0.0.1 <app-name>.loc`
5. Open the app via `http://<app-name>.loc:8080`
