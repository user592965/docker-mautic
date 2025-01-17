FROM php:7.4-fpm

LABEL vendor="Mautic"
LABEL maintainer="Luiz Eduardo Oliveira Fonseca <luiz@powertic.com>"

# Install PHP extensions
RUN apt-get update && apt-get install --no-install-recommends -y \
    cron \
    git \
    wget \
    sudo \
    libc-client-dev \
    libicu-dev \
    libkrb5-dev \
    libmcrypt-dev \
    libzip-dev \
    libssl-dev \
    libz-dev \
    libonig-dev \
    unzip \
    zip \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/* \
    && rm /etc/cron.daily/*

RUN pecl install mcrypt-1.0.3

RUN docker-php-ext-configure imap --with-imap --with-imap-ssl --with-kerberos \
    && docker-php-ext-configure opcache --enable-opcache \
    && docker-php-ext-install imap intl mbstring mysqli pdo_mysql zip opcache bcmath sockets \
    && docker-php-ext-enable imap intl mbstring mcrypt mysqli pdo_mysql zip opcache bcmath sockets

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

# Define Mautic volume to persist data
VOLUME /data

# Define Mautic version and expected SHA1 signature
ENV MAUTIC_VERSION 4.0.0-rc
ENV MAUTIC_SHA1 ded92ab7403f3c5c5ef0844b0933e0965502d7a5

# By default enable cron jobs
ENV MAUTIC_RUN_CRON_JOBS true

# Setting an root user for test
ENV MAUTIC_DB_USER root
ENV MAUTIC_DB_NAME mautic
ENV MAUTIC_DB_PORT 3306

# Default install configuration parameters
ENV MAUTIC_ADMIN_EMAIL root@example.org
ENV MAUTIC_ADMIN_PASSWORD password
ENV MAUTIC_URL localhost

# Setting PHP properties
ENV PHP_INI_DATE_TIMEZONE='UTC' \
    PHP_MEMORY_LIMIT=512M \
    PHP_MAX_UPLOAD=128M \
    PHP_MAX_EXECUTION_TIME=300
  
# Download package and extract to web volume
RUN curl -o /tmp/mautic.zip -SL https://github.com/mautic/mautic/releases/download/${MAUTIC_VERSION}/${MAUTIC_VERSION}.zip \
    && echo "$MAUTIC_SHA1 */tmp/mautic.zip" | sha1sum -c - \
    && rm -rf /var/www/html \
    && mkdir /var/www/html \
    && unzip /tmp/mautic.zip -d /var/www/html \
    && rm /tmp/mautic.zip \
    && chown -R www-data:www-data /var/www/html

# Drop in redirectors to volume
COPY common/paths_local.php /var/www/html/app/config/paths_local.php
COPY common/parameters_local.php /var/www/html/app/config/parameters_local.php

# Copy init scripts and custom .htaccess
COPY common/docker-entrypoint.sh /entrypoint.sh
COPY common/makeconfig.php /makeconfig.php
COPY common/makedb.php /makedb.php
COPY common/mautic.crontab /etc/cron.d/mautic
RUN chmod 644 /etc/cron.d/mautic

# Apply necessary permissions
RUN ["chmod", "+x", "/entrypoint.sh"]
ENTRYPOINT ["/entrypoint.sh"]

CMD ["php-fpm"]
