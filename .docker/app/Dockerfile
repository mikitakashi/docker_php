# #共通用のイメージができたら置き換え。一旦仮のbase image
FROM php:8.2-apache-buster as php_base
ENV TZ=Asia/Tokyo \
    LANG=C.UTF-8 \
    LANGUAGE=ja_JP \
    APP_ROOT=/srv/src \
    LIB_ROOT=/srv/lib \
    APP_USER=www-data \
    APP_GROUP=www-data
ENV DEPEND_LIBS libonig-dev libpng-dev libzip-dev
ENV DEPEND_LIBS_EXT gd opcache
WORKDIR ${APP_ROOT}
RUN apt-get update && apt-get install -y ${DEPEND_LIBS} \
    && apt-get purge -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-install ${DEPEND_LIBS_EXT} \
    && docker-php-ext-enable opcache


FROM php_base as php_dev
# install dev tools and add sudo to user
ENV DEV_TOOLS sudo git vim zip procps iputils-ping net-tools
RUN apt-get update && apt-get install -y ${DEV_TOOLS} \
    && pecl install xdebug-3.2.1 && docker-php-ext-enable xdebug \
    && mkdir -p /srv/log \
    && a2enmod rewrite
COPY --from=composer:2.5.8 /usr/bin/composer /usr/local/bin/composer


FROM php_base as php_serv
ENV DEV_TOOLS zip
RUN apt-get update && apt-get install -y ${DEV_TOOLS} \
    && apt-get purge -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p ${LIB_ROOT} \
    && mkdir -p /srv/log \
    && a2enmod rewrite
COPY --from=composer:2.5.8 /usr/bin/composer /usr/local/bin/composer
# copy sources
COPY --chown=${APP_USER}:${APP_USER} ./src ${APP_ROOT}
# copy configs
COPY  ./.docker/app/conf/php/php.ini /usr/local/etc/php/php.ini
COPY  ./.docker/app/conf/apache/conf/000-default.conf /etc/apache2/sites-enabled/000-default.conf
RUN composer install