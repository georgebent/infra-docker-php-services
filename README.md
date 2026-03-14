# Docker PHP Services

Docker-based infrastructure for running multiple PHP applications locally. It provides PHP-FPM 8.5, PostgreSQL 15, MongoDB, and Nginx as isolated services in a custom bridge network. Application source code lives in a sibling directory (`../services/`) that is volume-mounted into both the PHP and Nginx containers.

## Requirements

- Docker
- Docker Compose

## Setup

1. Copy `.env.example` to `.env` and adjust credentials if needed.
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
docker exec -it services_docker_php bash  # default when NAME=services_docker

# Run Composer inside the container
docker-compose exec php composer install --working-dir=/srv/src/<app-name>
```

## Architecture

```
docker-php-services/         ← this repo (infrastructure only)
../services/                 ← your PHP application code (mounted as /srv/src/)
```

Network: Custom bridge, subnet `172.20.0.0/16`

| Service     | Default container name     | IP           | Port mapping |
|-------------|----------------------------|--------------|--------------|
| PHP-FPM     | `services_docker_php`      | 172.20.0.10  | internal `9000` |
| PostgreSQL  | `services_docker_postgres` | 172.20.0.11  | `5432:5432` |
| MongoDB     | `services_docker_mongo`    | 172.20.0.12  | `27017:27017` |
| Nginx       | `services_docker_nginx`    | 172.20.0.30  | `8080:80` |

Request flow: browser → host `:8080` → Nginx `:80` → FastCGI `php:9000` → PostgreSQL

Nginx config files placed in `./etc/nginx/` are mounted directly to `/etc/nginx/conf.d/`. Add a new `.conf` file and restart Nginx to pick it up. Use `etc/nginx/example.conf` as a template and keep `fastcgi_pass php:9000`.

## PHP Container

- Base: `php:8.5-fpm`, runs as non-root user `www` (uid 1001)
- Web server image: `nginx:1.28.2-alpine`
- Extensions: `pdo`, `pdo_pgsql`, `pgsql`, `pdo_mysql`, `gd`, `zip`, `exif`, `pcntl`, `soap`, `xml`, `mbstring`, `intl`, `mongodb`, XDebug
- Composer available globally
- OPcache is configured for local development in `etc/php/config/opcache.ini`

## XDebug

Configured in `etc/php/config/xdebug.ini`:
- Mode: `develop,debug`
- Port: `9003`
- Client host: `host.docker.internal` (Docker Desktop). On Linux, `extra_hosts: host.docker.internal:host-gateway` in `docker-compose.yml` handles this.
- `PHP_IDE_CONFIG` is not set in `docker-compose.yml`; configure it in your IDE or container environment if your debugger setup needs a fixed server name.

## Adding a New Application

1. Place application code in `../services/<app-name>/`
2. Create `etc/nginx/<app-name>.conf` based on `etc/nginx/example.conf`
3. Set `server_name` to `<app-name>.loc` and `root` to `/srv/src/<app-name>/public`
4. Restart Nginx: `docker-compose restart nginx`
5. Add the domain to your `/etc/hosts`: `127.0.0.1 <app-name>.loc`
6. Open the app via `http://<app-name>.loc:8080`
