FROM php:7.3-fpm-alpine

# Project
ARG ENVIRONMENT=development
# PHP composer
ARG COMPOSER_PROCESS_TIMEOUT=3600
ARG COMPOSER_PARALLEL_INSTALL=1
ARG COMPOSER_JP_PACKAGIST=1
# PHP Xdebug
ARG PHP_XDEBUG_INSTALL=1
# PHP Xdebug
ARG PHP_XDEBUG_DEFAULT_ENABLE=on
ARG PHP_XDEBUG_COVERAGE_ENABLE=on
ARG PHP_XDEBUG_REMOTE_ENABLE=on
ARG PHP_XDEBUG_REMOTE_AUTO_START=on
ARG PHP_XDEBUG_REMOTE_CONNECT_BACK=off
ARG PHP_XDEBUG_REMOTE_HOST=host.docker.internal
ARG PHP_XDEBUG_REMOTE_PORT=9000
ARG PHP_XDEBUG_IDEKEY=PHPSTORM
ARG PHP_XDEBUG_PROFILER_ENABLE=off
ARG PHP_XDEBUG_PROFILER_OUTPUT_DIR=/tmp
# PHP-FPM Xdebug
ARG PHP_FPM_XDEBUG_DEFAULT_ENABLE=on
ARG PHP_FPM_XDEBUG_COVERAGE_ENABLE=on
ARG PHP_FPM_XDEBUG_REMOTE_ENABLE=on
ARG PHP_FPM_XDEBUG_REMOTE_AUTO_START=on
ARG PHP_FPM_XDEBUG_REMOTE_CONNECT_BACK=off
ARG PHP_FPM_XDEBUG_REMOTE_HOST=host.docker.internal
ARG PHP_FPM_XDEBUG_REMOTE_PORT=9000
ARG PHP_FPM_XDEBUG_IDEKEY=PHPSTORM
ARG PHP_FPM_XDEBUG_PROFILER_ENABLE=off
ARG PHP_FPM_XDEBUG_PROFILER_OUTPUT_DIR=/tmp

RUN apk update \
 && apk add --no-cache \
        bash \
        openssh-client \
        unzip \
        curl \
        git \
        libgsasl \
        libmemcached \
        freetype \
        libpng \
        libjpeg-turbo \
        freetype-dev \
        libpng-dev \
        jpeg-dev \
        libjpeg \
        libjpeg-turbo-dev \
#--------------------------------------------------------------------------
# PHP extensions
#--------------------------------------------------------------------------
 && apk add --no-cache --virtual .php-build-deps \
        autoconf \
        gcc \
        g++ \
        make \
        libc-dev \
        curl-dev \
        musl-dev \
        zlib-dev \
        libmemcached-dev \
        cyrus-sasl-dev \
 && pecl channel-update pecl.php.net \
#####################################
# MBString
# PDO
# OPcache
#####################################
 && NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
 && docker-php-ext-install -j${NPROC} \
     mbstring \
     pdo \
     pdo_mysql \
     opcache \
     exif \
#     zip \
 && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ \
 && docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) gd \
#####################################
# APCu
#####################################
 && pecl install -o -f apcu \
 && docker-php-ext-enable apcu \
#####################################
# PHP Memcached
#####################################
 && pecl install -o -f memcached \
 && docker-php-ext-enable memcached \
#####################################
# PHP Redis
#####################################
 && pecl install -o -f redis \
 && docker-php-ext-enable redis \
#####################################
# PHP Xdebug
#####################################
 && pecl install -o -f xdebug \
 && docker-php-ext-enable xdebug \
#####################################
# Modern XHProf
# compatible PHP Profiler
# for PHP 7
#####################################
 && git clone https://github.com/tideways/php-xhprof-extension \
 && cd php-xhprof-extension \
 && phpize \
 && ./configure \
 && make && make install \
 && cd .. \
 && rm -rf php-xhprof-extension \
 && docker-php-ext-enable tideways_xhprof \
#####################################
# PCOV
# ... under experiment ...
# Waiting for PHPUnit 8x
#####################################
####    && pecl install pcov \
####    && docker-php-ext-enable pcov \
#####################################
# PHP Composer
#####################################
 && curl -sS https://getcomposer.org/installer \
    | php -- --install-dir=/usr/local/bin --filename=composer \
#------
# done
#------
 && apk del .php-build-deps \
 && rm -rf /tmp/* /var/tmp/* \
#--------------------------------------------------------------------------
# timezone
#--------------------------------------------------------------------------
 && ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
#--------------------------------------------------------------------------
# user
#--------------------------------------------------------------------------
 && addgroup docker \
 && adduser -h /home/docker -s /bin/bash -G docker -D docker \
 && mkdir -p /home/docker/.ssh \
 && chown -R docker:docker /home/docker

#--------------------------------------------------------------------------
# PHP conf
#--------------------------------------------------------------------------
COPY ./conf.d /usr/local/etc/php/conf.d/

RUN set -ex; \
    { \
        echo "; Xdebug" ; \
        echo "xdebug.default_enable = ${PHP_XDEBUG_DEFAULT_ENABLE}" ; \
        echo "" ; \
        echo "; coverage" ; \
        echo "xdebug.coverage_enable = ${PHP_XDEBUG_COVERAGE_ENABLE}" ; \
        echo "" ; \
        echo "; remote debug" ; \
        echo "xdebug.remote_enable = ${PHP_XDEBUG_REMOTE_ENABLE}" ; \
        echo "xdebug.remote_autostart = ${PHP_XDEBUG_REMOTE_AUTO_START}" ; \
        echo "xdebug.remote_connect_back = ${PHP_XDEBUG_REMOTE_CONNECT_BACK}" ; \
        echo "xdebug.remote_host = ${PHP_XDEBUG_REMOTE_HOST}" ; \
        echo "xdebug.remote_port = ${PHP_XDEBUG_REMOTE_PORT}" ; \
        echo "xdebug.remote_handler = \"dbgp\"" ; \
        echo "xdebug.remote_mode = \"req\"" ; \
        echo "xdebug.remote_log = \"/proc/self/fd/2\"" ; \
        echo "xdebug.idekey = ${PHP_XDEBUG_IDEKEY}" ; \
        echo "" ; \
        echo "; profiler" ; \
        echo "xdebug.profiler_enable = ${PHP_XDEBUG_PROFILER_ENABLE}" ; \
        echo "xdebug.profiler_output_dir = ${PHP_XDEBUG_PROFILER_OUTPUT_DIR}" ; \
        echo "" ; \
    } >> /usr/local/etc/php/conf.d/z-aidiot-xdebug.ini

#--------------------------------------------------------------------------
# PHP-FPM conf
#--------------------------------------------------------------------------
COPY ./php-fpm.d /usr/local/etc/php-fpm.d/

RUN set -ex; \
    { \
        echo "" ; \
        echo "; Xdebug" ; \
        echo "php_admin_flag[xdebug.default_enable] = ${PHP_FPM_XDEBUG_DEFAULT_ENABLE}" ; \
        echo "" ; \
        echo "; Xdebug coverage" ; \
        echo "php_admin_flag[xdebug.coverage_enable] = ${PHP_FPM_XDEBUG_COVERAGE_ENABLE}" ; \
        echo "; Xdebug remote debug" ; \
        echo "php_admin_flag[xdebug.remote_enable] = ${PHP_FPM_XDEBUG_REMOTE_ENABLE}" ; \
        echo "php_admin_flag[xdebug.remote_autostart] = ${PHP_FPM_XDEBUG_REMOTE_AUTO_START}" ; \
        echo "php_admin_flag[xdebug.remote_connect_back] = ${PHP_FPM_XDEBUG_REMOTE_CONNECT_BACK}" ; \
        echo "php_admin_value[xdebug.remote_host] = ${PHP_FPM_XDEBUG_REMOTE_HOST}" ; \
        echo "php_admin_value[xdebug.remote_port] = ${PHP_FPM_XDEBUG_REMOTE_PORT}" ; \
        echo "php_admin_value[xdebug.idekey] = ${PHP_FPM_XDEBUG_IDEKEY}" ; \
        echo "" ; \
        echo "; Xdebug profiler" ; \
        echo "php_admin_flag[xdebug.profiler_enable] = ${PHP_FPM_XDEBUG_PROFILER_ENABLE}" ; \
        echo "php_admin_value[xdebug.profiler_output_dir] = ${PHP_FPM_XDEBUG_PROFILER_OUTPUT_DIR}" ; \
        echo "" ; \
    } >> /usr/local/etc/php-fpm.d/www.conf ;

#--------------------------------------------------------------------------
# Disabled Xdebug && Only PHP-FPM with Xdebug
#--------------------------------------------------------------------------
COPY ./php-fpm.ini /usr/local/etc/php/php-fpm.ini
RUN set -ex; \
    { \
        echo "" ; \
        cat /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini ; \
    } >> /usr/local/etc/php/php-fpm.ini ; \
    rm -f /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

#--------------------------------------------------------------------------
# docker-php-ext-disable
#--------------------------------------------------------------------------
COPY ./docker-php-ext-disable.sh /usr/local/bin/docker-php-ext-disable
RUN set -ex; chmod +x /usr/local/bin/docker-php-ext-disable

#--------------------------------------------------------------------------
# Environment
#--------------------------------------------------------------------------
VOLUME /var/www

ENV ENVIRONMENT ${ENVIRONMENT}
ENV COMPOSER_PROCESS_TIMEOUT ${COMPOSER_PROCESS_TIMEOUT}

USER docker

ENV TERM xterm
ENV LANG C.UTF-8

ENV HOME /home/docker

WORKDIR /var/www

# RUN set -ex; \
#     \
#     if [ "${COMPOSER_JP_PACKAGIST}" != "0" ]; then \
#         composer config -g repositories.packagist composer https://packagist.jp ; \
#     fi; \
#     \
#     if [ "${COMPOSER_PARALLEL_INSTALL}" != "0" ]; then \
#         composer global require hirak/prestissimo ; \
#     fi;

# Using SSH keys with GitLab
RUN set -ex; ssh-keyscan -t rsa github.com 2>&1 >> /home/docker/.ssh/known_hosts

# Let's Encrypt intermediate CA is not installed on the git.aidiot2.jp server
RUN set -ex; git config --global http.sslVerify false

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

EXPOSE 9000
CMD ["php-fpm", "--php-ini", "/usr/local/etc/php/php-fpm.ini"]
