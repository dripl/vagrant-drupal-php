FROM php:7.1-fpm-alpine

ENV GOTLP_VER 0.1.5
ENV RABBITMQ_C_VER 0.8.0
ENV WALTER_VER 1.3.0

ENV APP_ROOT /var/www/html

# Recreate user with correct params
RUN deluser www-data && \
	addgroup -g 82 -S www-data && \
	adduser -u 82 -D -S -s /bin/bash -G www-data www-data && \
	sed -i '/^www-data/s/!/*/' /etc/shadow

RUN set -xe && \
    apk add --update \
        bash \
        bzip2 \
        ca-certificates \
        c-client \
        git \
        gzip \
        icu-libs \
        imagemagick \
        imap \
        libbz2 \
        libjpeg-turbo \
        libmcrypt \
        libpng \
        libxslt \
        make \
        mariadb-client \
        libmemcached-libs \
        openssh \
        openssh-client \
        openssl \
        patch \
        postgresql-client \
        rsync \
        su-exec \
        tar \
        wget \
        yaml \

        # Temporarily packages to compile extensions
        autoconf \
        cmake \
        build-base \
        bzip2-dev \
        freetype-dev \
        icu-dev \
        imagemagick-dev \
        imap-dev \
        jpeg-dev \
        libjpeg-turbo-dev \
        libmemcached-dev \
        libmcrypt-dev \
        libpng-dev \
        libtool \
        libxslt-dev \
        openldap-dev \
        pcre-dev \
        postgresql-dev \
        yaml-dev && \

    docker-php-source extract && \

    docker-php-ext-install \
        bcmath \
        bz2 \
        calendar \
        exif \
        imap \
        intl \
        ldap \
        mcrypt \
        mysqli \
        opcache \
        pdo_mysql \
        pdo_pgsql \
        pgsql \
        phar \
        soap \
        sockets \
        xmlrpc \
        xsl \
        zip && \

    # GD
    docker-php-ext-configure gd \
        --with-gd \
        --with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ && \
      NPROC=$(getconf _NPROCESSORS_ONLN) && \
      docker-php-ext-install -j${NPROC} gd && \

    # RabbitMQ C client
    wget -qO- https://github.com/alanxz/rabbitmq-c/releases/download/v${RABBITMQ_C_VER}/rabbitmq-c-${RABBITMQ_C_VER}.tar.gz \
        | tar xz -C /tmp/ && \
    cd /tmp/rabbitmq-c-${RABBITMQ_C_VER} && \
    mkdir -p build && cd build && \
    cmake .. \
    		-DCMAKE_INSTALL_PREFIX=/usr \
    		-DCMAKE_INSTALL_LIBDIR=lib \
    		-DCMAKE_C_FLAGS="$CFLAGS" && \
    cmake --build . --target install && \

    # PECL extensions
    pecl config-set php_ini "$PHP_INI_DIR/php.ini" && \

    pecl install \
        amqp \
        apcu \
        oauth \
        imagick \
        mongodb \
        redis \
        xdebug \
        yaml-2.0.0 && \

    docker-php-ext-enable \
        amqp \
        apcu \
        oauth \
        imagick \
        mongodb \
        redis \
        xdebug \
        yaml && \

    # Uploadprogress
    git clone https://github.com/php/pecl-php-uploadprogress/ /usr/src/php/ext/uploadprogress/ && \
    docker-php-ext-configure uploadprogress && \
    docker-php-ext-install uploadprogress && \

    # Memcached
    git clone --branch php7 https://github.com/php-memcached-dev/php-memcached /usr/src/php/ext/memcached/ && \
    docker-php-ext-configure memcached && \
    docker-php-ext-install memcached && \

    # AST
    git clone https://github.com/nikic/php-ast /usr/src/php/ext/ast/ && \
    docker-php-ext-configure ast && \
    docker-php-ext-install ast && \

    # Download and install Gotlp
    wget -qO- https://github.com/wodby/gotpl/releases/download/${GOTLP_VER}/gotpl-alpine-linux-amd64-${GOTLP_VER}.tar.gz \
        | tar xz -C /usr/local/bin && \

    # Install composer
    wget -qO- https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \

    # Install composer parallel install plugin
    su-exec www-data composer global require hirak/prestissimo:^0.3 && \

    # Install Walter tool
    wget -qO- https://github.com/walter-cd/walter/releases/download/v${WALTER_VER}/walter_${WALTER_VER}_linux_amd64.tar.gz \
        | tar xz -C /tmp/ && \
    mv /tmp/walter_linux_amd64/walter /usr/local/bin && \

    # Disable Xdebug by default
    rm -f $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini && \

    # Cleanup
    su-exec www-data composer clear-cache && \
    docker-php-source delete && \
    apk del --purge \
        *-dev \
        autoconf \
        build-base \
        cmake \
        libtool && \

    rm -rf \
        /usr/src/php.tar.xz \
        /usr/src/php/ext/memcached \
        /usr/src/php/ext/uploadprogress \
        /usr/include/php \
        /usr/lib/php/build \
        /var/cache/apk/* \
        /tmp/* \
        /root/.composer

ENV PATH "/home/www-data/.composer/vendor/bin:$PATH"

WORKDIR $APP_ROOT
EXPOSE 9000

COPY templates/* /etc/gotpl/
COPY docker-entrypoint.sh /
COPY actions /usr/local/bin

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["php-fpm"]
