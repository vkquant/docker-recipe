FROM php:7.4-fpm

# Set working directory
WORKDIR /var/www/project

RUN apt-get update && apt-get install -y \
    zlib1g-dev \
    libzip-dev \
    libpq-dev \
    libpng-dev \
    libonig-dev \
    libicu-dev \
    libxml2-dev \
    git \
    ssh

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install \
    mysqli \
    pdo \
    pdo_mysql \
    mbstring \
    tokenizer \
    intl \
    xml \
    ctype \
    json \
    zip \
    gd

COPY --from=composer:2.0.4 /usr/bin/composer /usr/bin/composer

RUN curl -L https://download.newrelic.com/php_agent/archive/9.14.0.290/newrelic-php5-9.14.0.290-linux.tar.gz | tar -C /tmp -zx \
    && export NR_INSTALL_USE_CP_NOT_LN=1 \
    && export NR_INSTALL_SILENT=1 \
    && /tmp/newrelic-php5-9.14.0.290-linux/newrelic-install install \
    && rm -rf /tmp/newrelic-php5-* /tmp/nrinstall*

RUN sed -i -e "s/REPLACE_WITH_REAL_KEY/" \
    -e "s/newrelic.appname[[:space:]]=[[:space:]].*/newrelic.appname=\"${NEW_RELIC_APPNAME}\"/" \
    -e '$anewrelic.distributed_tracing_enabled=true' \
    -e '$anewrelic.daemon.address="newrelic-php-daemon:31339"' \
    $(php -r "echo(PHP_CONFIG_FILE_SCAN_DIR);")/newrelic.ini

RUN mkdir -p /root/.ssh
COPY ./docker/php/keys/id_rsa /root/.ssh/id_rsa
COPY ./docker/php/keys/id_rsa.pub /root/.ssh/id_rsa.pub

COPY ./composer.json /var/www/project
COPY ./composer.lock /var/www/project

RUN chmod 600 /root/.ssh/id_rsa.pub
RUN chmod 600 /root/.ssh/id_rsa
RUN ssh-keyscan bitbucket.org > /root/.ssh/known_hosts
RUN ssh -T git@bitbucket.org
RUN ssh -v git@bitbucket.org

COPY . /var/www/project

#### Build application #### Only for build images
RUN composer install --no-cache

CMD php artisan view:clear \
    && php artisan cache:clear \
    && php artisan config:clear \
    && php artisan migrate
#    && php artisan migrate --seed


