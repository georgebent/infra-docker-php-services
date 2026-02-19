# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Docker-based infrastructure for running multiple PHP applications locally. It provides PHP-FPM 8.4, PostgreSQL 15, and Nginx as isolated services in a custom bridge network. Application source code lives in a **sibling directory** (`../services/`) that is volume-mounted into both the PHP and Nginx containers.

## Setup

1. Copy `.env.example` to `.env` and fill in `POSTGRES_DB` (and adjust credentials if needed)
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

# Run Composer inside the container
docker-compose exec php composer install --working-dir=/srv/src/<app-name>
```

## Architecture

```
docker-php-services/         ← this repo (infrastructure only)
../services/                 ← your PHP application code (mounted as /srv/src/)
```

**Network:** Custom bridge, subnet `172.10.1.0/16`

| Service    | Container name              | IP            | Port |
|------------|-----------------------------|---------------|------|
| PHP-FPM    | `services_docker_php`       | 172.10.1.10   | 9000 |
| PostgreSQL  | `services_docker_postgres`  | 172.10.1.11   | 5432 |
| Nginx      | `services_docker_nginx`     | 172.10.1.30   | 80   |

**Request flow:** Nginx (port 80) → FastCGI → `services_docker_php:9000` → PostgreSQL

Nginx config files placed in `./etc/nginx/` are mounted directly to `/etc/nginx/conf.d/`, so adding a new `.conf` file is picked up on container restart without a rebuild. Use `etc/nginx/example.conf` as the template for new virtual hosts — set `server_name`, `root`, and keep the `fastcgi_pass services_docker_php:9000` directive.

## PHP Container

- Base: `php:8.4-fpm`, runs as non-root user `www` (uid 1001)
- PHP extensions: `pdo`, `pdo_pgsql`, `pgsql`, `pdo_mysql`, `gd`, `zip`, `exif`, `pcntl`, `soap`, `xml`, XDebug
- Composer available globally

## XDebug

XDebug 3 is pre-installed and configured in `etc/php/config/xdebug.ini`:
- Mode: `develop,debug`
- Port: `9003`
- Client host: `host.docker.internal` (works with Docker Desktop; on Linux the `extra_hosts: host.docker.internal:host-gateway` entry in docker-compose.yml handles this)
- IDE server name: `services.loc` (set via `PHP_IDE_CONFIG` env var)

## Adding a New Application

1. Place application code in `../services/<app-name>/`
2. Create `etc/nginx/<app-name>.conf` based on `etc/nginx/example.conf`, adjusting `server_name` and `root`
3. Restart Nginx: `docker-compose restart nginx`
4. Add the domain to your `/etc/hosts`: `127.0.0.1 <app-name>.loc`
