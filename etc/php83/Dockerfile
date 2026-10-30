# PHP 8.3 to match the production hosting of the Laravel sites
FROM php:8.3-fpm

WORKDIR /srv/src

RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libicu-dev \
    libonig-dev \
    libzip-dev \
    libxml2-dev \
    zip \
    unzip \
    git \
    curl \
    nano

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd pdo_mysql zip exif pcntl intl mbstring xml

RUN apt-get install -y $PHPIZE_DEPS \
    && pecl install xdebug \
    && docker-php-ext-enable xdebug

RUN groupadd -g 1001 www
RUN useradd -u 1001 -ms /bin/bash -g www www

USER www
