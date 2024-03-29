FROM php:8.2.7-fpm-alpine

ARG USER=tester
ARG ENV=dev

WORKDIR /app

# CREATE USER AND INSTALL APK PACKAGES
RUN adduser -s /bin/sh -D ${USER} \
  && chown -R ${USER}:${USER} /app \
  && apk update \
  && apk add --no-cache curl zip unzip nginx supervisor

# INSTALL COMPOSER
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php \
  && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
  && rm -rf composer-setup.php \
  && mkdir /.composer \
  && chmod -R ugo+rw /.composer

# INSTALL PHP EXTENTIONS
ADD https://github.com/mlocati/docker-php-extension-installer/releases/download/2.1.15/install-php-extensions /usr/local/bin/

RUN chmod uga+x /usr/local/bin/install-php-extensions && sync \
  && install-php-extensions pdo pdo_mysql gd bcmath csv memcache zip rdkafka

# CONFIGURE NGINX
COPY laravel-app/docker/nginx.conf /etc/nginx/nginx.conf
RUN touch /var/run/nginx.pid \
  && chown -R ${USER}:${USER} /var/log/nginx \
  && chown -R ${USER}:${USER} /var/lib/nginx \
  && chown -R ${USER}:${USER} /var/run/nginx.pid

# INSTALL START-CONTAINER SCRIPT
COPY laravel-app/docker/start-${ENV} /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

# CONFIGURE PHP-FPM
COPY laravel-app/docker/php-fpm.conf /usr/local/etc/php-fpm.d/www.conf
COPY laravel-app/docker/php.ini /usr/local/etc/php/conf.d/app.ini
RUN mkdir /var/log/php \
  && chown -R ${USER}:${USER} /var/log/php

# CONFIGURE SUPERVISOR
COPY laravel-app/docker/supervisor.conf /etc/supervisord.conf

USER ${USER}

EXPOSE 80
CMD ["start-container"]
