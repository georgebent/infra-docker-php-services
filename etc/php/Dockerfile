FROM php:8.4-fpm

# Set working directory
WORKDIR /srv/src

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libicu-dev \
    libonig-dev \
    locales \
    zip \
    libzip-dev\
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    libxml2-dev \
    curl \
    mc \
    nano

# Install Postgre PDO
RUN apt-get install -y libpq-dev \
    && docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
    && docker-php-ext-install pdo pdo_pgsql pgsql

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install extensions
RUN docker-php-ext-install pdo_mysql zip exif pcntl
RUN docker-php-ext-install soap
RUN docker-php-ext-configure intl && docker-php-ext-install intl
RUN docker-php-ext-install mbstring opcache

RUN apt-get install -y libssl-dev \
    && pecl install mongodb \
    && docker-php-ext-enable mongodb

RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

# Add user
RUN groupadd -g 1001 www
RUN useradd -u 1001 -ms /bin/bash -g www www

USER www
