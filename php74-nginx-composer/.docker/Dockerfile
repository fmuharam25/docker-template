ARG ALPINE_VERSION
ARG APP_PORT

FROM alpine:${ALPINE_VERSION}

LABEL Maintainer="Fauzan M Muharam" \
      Description="Lightweight container with Nginx & PHP-FPM 7.4 based on Alpine Linux (forked from trafex/alpine-nginx-php7)."

# make sure you can use HTTPS
RUN apk --update add ca-certificates

# Install packages
RUN apk --no-cache add php php-fpm php-opcache php-openssl php-curl \
    php-json php-phar php-mbstring php-session php-exif php-gd php-pdo php-pdo_mysql php-pdo_sqlite php-mysqli\
    php-bcmath php-calendar php-intl php-pcntl php-tokenizer php-xml php-zip php-dom\
    nginx supervisor curl mysql-client zlib-dev git

# https://github.com/codecasts/php-alpine/issues/21
#RUN ln -s /usr/bin/php7 /usr/bin/php


# Remove default server definition
#RUN rm /etc/nginx/conf.d/default.conf

# Setup document root
RUN mkdir -p /var/www/html
COPY ./src /var/www/html

#Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody.nobody /var/www/html && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx 

RUN chmod o+w /var/www/html/storage -R

# Switch to use a non-root user from here on
USER nobody

# Add application
WORKDIR /var/www/html

RUN composer install --ignore-platform-reqs && php artisan key:generate
RUN composer dump-autoload --optimize && php artisan config:clear

# Expose the port nginx is reachable on
EXPOSE ${APP_PORT}

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:${APP_PORT}/fpm-ping
