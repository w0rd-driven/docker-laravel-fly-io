FROM ubuntu:22.04

LABEL maintainer="Jeremy Brayton"

ARG WWWGROUP=1000
ARG WWWUSER=1000
ARG NODE_VERSION=16
ARG POSTGRES_VERSION=14

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Setup and install PHP
RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 \
    && mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && echo "keyserver hkp://keyserver.ubuntu.com:80" >> ~/.gnupg/dirmngr.conf \
    && gpg --recv-key 0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c \
    && gpg --export 0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c > /usr/share/keyrings/ppa_ondrej_php.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php8.1-cli php8.1-fpm php8.1-dev \
       php8.1-pgsql php8.1-sqlite3 php8.1-gd \
       php8.1-curl \
       php8.1-imap php8.1-mysql php8.1-mbstring \
       php8.1-xml php8.1-zip php8.1-bcmath php8.1-soap \
       php8.1-intl php8.1-readline \
       php8.1-ldap \
       php8.1-msgpack php8.1-igbinary php8.1-redis php8.1-swoole \
       php8.1-memcached php8.1-pcov php8.1-xdebug

# Nginx
RUN apt-get install -y nginx

# Composer
RUN php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash -

# Node
RUN apt-get install -y nodejs \
    && npm install -g npm \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /usr/share/keyrings/yarn.gpg >/dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && curl -sS https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | tee /usr/share/keyrings/pgdg.gpg >/dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt jammy-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && apt-get update \
    && apt-get install -y yarn

# Database Clients
RUN apt-get install -y mysql-client \
    && apt-get install -y postgresql-client-$POSTGRES_VERSION

# Utilities
RUN apt-get install -y htop

# Cleanup
RUN apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.1

RUN groupadd --force -g $WWWGROUP laravel
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 laravel

# Setup container entrypoint
COPY start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

# Setup Supervisor
COPY etc/supervisor/conf.d/docker.conf /etc/supervisor/conf.d/docker.conf
# COPY etc/supervisor/conf.d/laravel-horizon.conf /etc/supervisor/conf.d/laravel-horizon.conf
# COPY etc/supervisor/conf.d/laravel-notifications.conf /etc/supervisor/conf.d/laravel-notifications.conf
# COPY etc/supervisor/conf.d/laravel-queue.conf /etc/supervisor/conf.d/laravel-queue.conf
# COPY etc/supervisor/conf.d/laravel-schedule.conf /etc/supervisor/conf.d/laravel-schedule.conf
COPY etc/supervisor/conf.d/nginx.conf /etc/supervisor/conf.d/nginx.conf
COPY etc/supervisor/conf.d/php-fpm.conf /etc/supervisor/conf.d/php-fpm.conf
COPY etc/supervisor/conf.d/group.conf /etc/supervisor/conf.d/group.conf

# Setup PHP
COPY php.ini /etc/php/8.1/cli/conf.d/99-laravel.ini

# Setup Nginx
# RUN cp /etc/nginx/nginx.conf /etc/nginx/nginx.old.conf && cp /etc/nginx/http.d/default.conf /etc/nginx/http.d/default.old.conf

# Setup Composer
ENV COMPOSER_HOME /.composer
ENV COMPOSER_CACHE_DIR /cache
ENV PATH /.composer/vendor/bin:$PATH

EXPOSE 8000

ENTRYPOINT ["start-container"]
