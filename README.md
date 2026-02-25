# Docker PHP Services

Docker-based infrastructure for running multiple PHP applications locally. It provides PHP-FPM 8.4, PostgreSQL 15, MongoDB, and Nginx as isolated services in a custom bridge network. Application source code lives in a sibling directory (`../services/`) that is volume-mounted into both the PHP and Nginx containers.

## Requirements

- Docker
- Docker Compose

## Setup

1. Copy `.env.example` to `.env` and fill in `POSTGRES_DB` (adjust credentials if needed).
2. Start services:
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

Network: Custom bridge, subnet `172.10.1.0/16`

| Service     | Container name             | IP           | Port  |
|-------------|----------------------------|--------------|-------|
| PHP-FPM     | `services_docker_php`      | 172.10.1.10  | 9000  |
| PostgreSQL  | `services_docker_postgres` | 172.10.1.11  | 5432  |
| MongoDB     | `services_docker_mongo`    | 172.10.1.12  | 27017 |
| Nginx       | `services_docker_nginx`    | 172.10.1.30  | 80    |

Request flow: Nginx (port 80) → FastCGI → `services_docker_php:9000` → PostgreSQL

Nginx config files placed in `./etc/nginx/` are mounted directly to `/etc/nginx/conf.d/`. Add a new `.conf` file and restart Nginx to pick it up. Use `etc/nginx/example.conf` as a template and keep `fastcgi_pass services_docker_php:9000`.

## PHP Container

- Base: `php:8.4-fpm`, runs as non-root user `www` (uid 1001)
- Extensions: `pdo`, `pdo_pgsql`, `pgsql`, `pdo_mysql`, `gd`, `zip`, `exif`, `pcntl`, `soap`, `xml`, `mongodb`, XDebug
- Composer available globally

## XDebug

Configured in `etc/php/config/xdebug.ini`:
- Mode: `develop,debug`
- Port: `9003`
- Client host: `host.docker.internal` (Docker Desktop). On Linux, `extra_hosts: host.docker.internal:host-gateway` in `docker-compose.yml` handles this.
- IDE server name: `services.loc` (set via `PHP_IDE_CONFIG`)

## Adding a New Application

1. Place application code in `../services/<app-name>/`
2. Create `etc/nginx/<app-name>.conf` based on `etc/nginx/example.conf`
3. Restart Nginx: `docker-compose restart nginx`
4. Add the domain to your `/etc/hosts`: `127.0.0.1 <app-name>.loc`
