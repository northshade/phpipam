FROM alpine:3.11
LABEL Maintainer="Shad Northrop" \
      Description="PHPIPAM on Alpine linux with NGINX."

ENV PHPIPAM_SOURCE https://github.com/northshade/
ENV PHPIPAM_VERSION 1.4
ENV WEB_REPO /var/www/html
ENV IPAM_BASE /
ENV IPAM_DATABASE_HOST db
ENV IPAM_DATABASE_USER phpipam
ENV IPAM_DATABASE_PASS CHANGEME
ENV IPAM_DATABASE_NAME phpipam
ENV IPAM_DATABASE_PORT 3306
ENV IPAM_DATABASE_WEBHOST=""
ENV IPAM_GMAPS_API_KEY=""
ENV PROXY_ENABLED=""
ENV PROXY_SERVER=""
ENV PROXY_PORT=""
ENV PROXY_USER=""
ENV PROXY_PASS=""
ENV PROXY_USE_AUTH=""


# Install packages
RUN apk --no-cache add php7 \
  php7-fpm \
  php7-pdo \
  php7-pdo_mysql \
  php7-mysqli \
  php7-json \
  php7-openssl \
  php7-curl \
  php7-zlib \
  php7-xml \
  php7-phar \
  php7-intl \
  php7-dom \
  php7-xmlreader \
  php7-ctype \
  php7-session \
  php7-sockets \
  php7-mbstring \
  php7-gd \
  php7-gmp \
  php7-ldap \
  php7-mcrypt \
  php7-simplexml \
  php7-gettext \
  php7-pcntl \
  php7-pear \
  php7-cli \
  php7-iconv \
  php7-snmp \
  nginx \
  supervisor \
  curl \
  git

# Configure nginx
COPY php-nginx/config/nginx.conf /etc/nginx/nginx.conf
# Remove default server definition
RUN rm /etc/nginx/conf.d/default.conf

# Configure PHP-FPM
COPY php-nginx/config/fpm-pool.conf /etc/php7/php-fpm.d/www.conf
COPY php-nginx/config/php.ini /etc/php7/conf.d/custom.ini

# Configure supervisord
COPY php-nginx/config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Setup document root
RUN mkdir -p /var/www/html

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody.nobody /var/www/html && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx

WORKDIR ${WEB_REPO}
RUN rm -f * && \
    git clone ${PHPIPAM_SOURCE}phpipam.git ${WEB_REPO} && \
    git checkout ${PHPIPAM_VERSION} && \
    git submodule update --init --recursive

# Use system environment variables into config.php
RUN ln -s config.docker.php config.php

USER nobody
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping