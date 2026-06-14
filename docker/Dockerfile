FROM php:8.3-apache

RUN apt-get update && apt-get install -y \
    libcurl4-openssl-dev libicu-dev libpng-dev libzip-dev libxml2-dev libonig-dev \
    zip unzip git curl

RUN docker-php-ext-install mysqli pdo_mysql curl intl mbstring gd zip soap opcache

RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

RUN a2enmod rewrite

ENV APACHE_DOCUMENT_ROOT=/var/www/moodle/public

RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' \
    /etc/apache2/sites-available/*.conf \
    /etc/apache2/apache2.conf \
    /etc/apache2/conf-available/*.conf
