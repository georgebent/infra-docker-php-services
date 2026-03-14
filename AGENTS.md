# AGENTS.md

## Session Convention
- Do not execute any terminal/shell commands in this project unless the user explicitly asks for it in the current conversation.

## What This Is
- This repository contains Docker-based local infrastructure for multiple PHP applications.
- Application source code does not live in this repo. It lives in the sibling directory `../services/` and is volume-mounted into the PHP and Nginx containers as `/srv/src/`.

## Setup
1. Copy `.env.example` to `.env`.
2. Fill in `POSTGRES_DB` and adjust credentials if needed.
3. Start services with `docker-compose up --build`.

## Common Commands
- Start services detached: `docker-compose up -d`
- Rebuild the PHP container after Dockerfile changes: `docker-compose up --build php`
- View logs: `docker-compose logs -f php` or `docker-compose logs -f nginx`
- Open a shell in the PHP container: `docker-compose exec php bash`
- Alternative PHP container shell: `docker exec -it services_docker_php bash`
- Run Composer inside the container: `docker-compose exec php composer install --working-dir=/srv/src/<app-name>`

## Architecture
- Repo layout:
  - `docker-php-services/`: infrastructure only
  - `../services/`: PHP application code
- Network: custom bridge subnet `172.10.1.0/16`
- Services:
  - PHP-FPM: `services_docker_php`, `172.10.1.10`, port `9000`
  - PostgreSQL: `services_docker_postgres`, `172.10.1.11`, port `5432`
  - MongoDB: `services_docker_mongo`, `172.10.1.12`, port `27017`
  - Nginx: `services_docker_nginx`, `172.10.1.30`, port `80`
- Request flow: Nginx -> FastCGI -> `services_docker_php:9000` -> PostgreSQL

## Nginx Conventions
- Nginx config files in `./etc/nginx/` are mounted to `/etc/nginx/conf.d/`.
- Adding a new `.conf` file is picked up on container restart and does not require a rebuild.
- Use `etc/nginx/example.conf` as the template for new virtual hosts.
- Set `server_name`, `root`, and keep `fastcgi_pass services_docker_php:9000`.

## PHP Container
- Base image: `php:8.5-fpm`
- Runs as non-root user `www` with uid `1001`
- Installed extensions: `pdo`, `pdo_pgsql`, `pgsql`, `pdo_mysql`, `gd`, `zip`, `exif`, `pcntl`, `soap`, `xml`, `mbstring`, `intl`, `mongodb`, `xdebug`
- Composer is available globally
- OPcache is configured for local development in `etc/php/config/opcache.ini`

## XDebug
- XDebug 3 is pre-installed and configured in `etc/php/config/xdebug.ini`
- Mode: `develop,debug`
- Port: `9003`
- Client host: `host.docker.internal`
- On Linux, `docker-compose.yml` maps `host.docker.internal:host-gateway`
- IDE server name: `services.loc` via `PHP_IDE_CONFIG`

## Adding a New Application
1. Place application code in `../services/<app-name>/`.
2. Create `etc/nginx/<app-name>.conf` from `etc/nginx/example.conf`.
3. Adjust `server_name` and `root`.
4. Restart Nginx with `docker-compose restart nginx`.
5. Add `127.0.0.1 <app-name>.loc` to `/etc/hosts`.
