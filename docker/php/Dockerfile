FROM php:7.4-fpm-alpine
COPY ./src /work
COPY ./docker/php/php.ini /usr/local/etc/php/php.ini
LABEL maintainer="LPB-Studio"

SHELL ["/bin/ash", "-oeux", "pipefail", "-c"]

WORKDIR /work

RUN apk add --no-cache bash
RUN chown -R www-data:www-data /work

# tinker(psysh)
ARG PSYSH_DIR=/usr/local/share/psysh
ARG PSYSH_PHP_MANUAL=$PSYSH_DIR/php_manual.sqlite
ARG PHP_MANUAL_URL=http://psysh.org/manual/ja/php_manual.sqlite

# timezone environment
ENV TZ=Asia/Tokyo
# locale
ENV LANG=ja_JP.UTF-8
# JSTに変更
RUN apk add tzdata && \
    cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

RUN apk upgrade --update && \
  apk --no-cache add icu-dev autoconf make g++ gcc ca-certificates wget curl

# for gd
RUN apk add --no-cache \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev

# for imap
RUN apk add imap-dev krb5-dev libressl-dev

# 使用するモジュール追加
RUN apk add --no-cache php7-bcmath php7-pecl-memcached zip unzip libzip-dev git
RUN docker-php-ext-install gd intl pdo_mysql opcache bcmath

# for imap
RUN PHP_OPENSSL=yes \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    && docker-php-ext-enable imap

# for libreoffice
RUN mkdir -p /usr/share/man/man1
RUN apk add --no-cache libreoffice

## Deploy Laravel Libs by Composer
COPY ./src/composer.json /tmp/composer.json
COPY ./src/composer.lock /tmp/composer.lock
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV PATH $PATH:/composer/vendor/bin
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN cp .env.example .env
RUN composer install --no-scripts --no-autoloader -d /tmp
RUN mv -n /tmp/vendor ./ \
    && composer dump-autoload
RUN php artisan key:generate
RUN php artisan cache:clear
RUN php artisan config:clear
RUN php artisan route:clear
RUN php artisan view:clear
RUN php artisan clear-compiled

## Laravel Permission
RUN chmod -R a+w storage/ bootstrap/cache
RUN chown -R www-data:www-data storage
RUN chmod -R 775 storage