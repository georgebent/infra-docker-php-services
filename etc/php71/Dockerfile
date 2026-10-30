# Legacy PHP 7.1 for running the old Laravel 5.x sites before their upgrade
FROM php:7.1-fpm

WORKDIR /srv/src

# Debian buster is archived, point apt at archive.debian.org
RUN sed -i -e 's|deb.debian.org|archive.debian.org|g' \
        -e 's|security.debian.org|archive.debian.org|g' \
        -e '/buster-updates/d' /etc/apt/sources.list

RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libzip-dev \
    libxml2-dev \
    zip \
    unzip \
    git \
    curl

# Composer 2.2 LTS is the last line that supports PHP 7.1
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --2.2

# Laravel < 5.8.35 cannot read the Composer 2 installed.json format, so keep Composer 1 for such sites
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer1 --1

RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ \
    && docker-php-ext-install gd pdo_mysql zip exif mbstring xml

# Add user
RUN groupadd -g 1001 www
RUN useradd -u 1001 -ms /bin/bash -g www www

USER www
