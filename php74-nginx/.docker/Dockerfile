ARG ALPINE_VERSION
ARG APP_PORT

FROM alpine:${ALPINE_VERSION}

LABEL Maintainer="Fauzan M Muharam"

# make sure you can use HTTPS
RUN apk --update add ca-certificates

# Install packages
RUN apk --no-cache add php php-fpm php-opcache php-openssl php-curl \
    nginx supervisor curl mysql-client zlib-dev git

# Setup document root
RUN mkdir -p /var/www/html
COPY ./src /var/www/html

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody.nobody /var/www/html && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx 

# Switch to use a non-root user from here on
USER nobody

# Add application
WORKDIR /var/www/html

# Expose the port nginx is reachable on
EXPOSE ${APP_PORT}

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:${APP_PORT}/fpm-ping
