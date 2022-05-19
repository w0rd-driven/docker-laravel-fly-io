FROM php:8.1-fpm

LABEL Maintainer="Jeremy Brayton <jeremy@braytonium.com>"

# apt-get required packages
RUN apt-get update -yqq && apt-get install -yqq git openssh-client libzip-dev zip unzip iputils-ping libicu-dev libxml2-dev

# PHP extension installation
RUN docker-php-ext-install pdo_mysql zip bcmath pcntl posix intl soap

# Install iconv and gd
RUN apt-get install -yqq libfreetype6-dev libjpeg62-turbo-dev libpng-dev \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# Install xdebug
#RUN pecl install xdebug \
#    && docker-php-ext-enable xdebug

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Environmental variables
ENV COMPOSER_HOME /root/.composer
ENV COMPOSER_CACHE_DIR /cache
ENV PATH /root/.composer/vendor/bin:$PATH
