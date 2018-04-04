# PHP Docker image for Yii 2.0 Framework runtime
# ==============================================

ARG PHP_BASE_IMAGE_VERSION
FROM php:${PHP_BASE_IMAGE_VERSION}

ENV PHP_MONGO_VERSION 1.4.2
ENV LIBRARY_PATH=/lib:/usr/lib

RUN apk update && \
    apk --update --no-cache --virtual .build-deps add \
        autoconf \
        cmake \
        curl-dev \
        file \
        freetype-dev \
        g++ \
        gcc \
        icu-dev \
        imagemagick-dev \
        libc-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libtool \
        libxml2-dev \
        make \
        pcre-dev \
        pkgconf \
        re2c \
        zlib-dev && \
    apk add \
        git \
        curl \
        bash \
        bash-completion \
        icu \
        imagemagick \
        pcre \
        freetype \
        libintl \
        libjpeg-turbo \
        libltdl \
        libpng \
        libxml2 \
        wget && \
    docker-php-ext-configure gd \
        --with-gd \
        --with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-configure bcmath && \
    docker-php-ext-install \
        soap \
        zip \
        curl \
        bcmath \
        exif \
        gd \
        iconv \
        intl \
        mbstring \
        opcache && \
    pecl install \
        imagick

# Install and compile MongoDB php driver
RUN git clone --branch ${PHP_MONGO_VERSION} https://github.com/mongodb/mongo-php-driver /tmp/php-mongo && \
    cd /tmp/php-mongo && \
    git submodule sync && git submodule update --init && \
    phpize && \
    ./configure && \
    make && \
    make install && \
    make test && \
    rm -rf /tmp/*

RUN apk del .build-deps

RUN echo "extension=imagick.so" > /usr/local/etc/php/conf.d/pecl-imagick.ini


# Configure version constraints
ENV PHP_ENABLE_XDEBUG=0 \
    PATH=/app:/app/vendor/bin:/root/.composer/vendor/bin:$PATH \
    TERM=linux \
    VERSION_PRESTISSIMO_PLUGIN=^0.3.7 \
    COMPOSER_ALLOW_SUPERUSER=1

# Add configuration files
COPY image-files/ /

# Add GITHUB_API_TOKEN support for composer
RUN chmod 700 \
        /usr/local/bin/docker-php-entrypoint \
        /usr/local/bin/composer

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- \
        --filename=composer.phar \
        --install-dir=/usr/local/bin && \
    composer clear-cache

# Install composer plugins
RUN composer global require --optimize-autoloader \
        "hirak/prestissimo:${VERSION_PRESTISSIMO_PLUGIN}" && \
    composer global dumpautoload --optimize && \
    composer clear-cache

# Application environment
WORKDIR /app

CMD ["php-fpm"]