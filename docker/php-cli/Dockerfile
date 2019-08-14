ARG TARGET_PHP_VERSION=7.3.8
ARG TARGET_ALPINE_VERSION=3.10

FROM php:${TARGET_PHP_VERSION}-cli-alpine${TARGET_ALPINE_VERSION}
ENV PHP_MONGO_VERSION 1.5.3
ENV PHP_REDIS_VERSION 5.0.5

# persistent / runtime deps
ENV PHPIZE_DEPS \
    autoconf \
    cmake \
    file \
    g++ \
    gcc \
    libc-dev \
    pcre-dev \
    make \
    git \
    pkgconf \
    re2c \
    # for GD
    freetype-dev \
    libpng-dev  \
    libjpeg-turbo-dev
ENV PHP_EXTENSIONS \
    gd \
    bcmath \
    intl \
    pcntl \
    mysqli \
    pdo_mysql \
    pdo_pgsql \
    mbstring \
    soap \
    iconv \
    opcache
ENV PECL_EXT \
    redis \
    apcu \
    mongodb

RUN apk add --no-cache --virtual .persistent-deps \
    # for intl extension
    icu-dev \
    # for postgres
    postgresql-dev \
    # for soap
    libxml2-dev \
    # for amqp
    libressl-dev \
    # for GD
    freetype \
    libpng \
    libjpeg-turbo
RUN set -xe \
    # workaround for rabbitmq linking issue
    && ln -s /usr/lib /usr/local/lib64 \
    && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
    && docker-php-ext-configure gd \
        --with-gd \
        --with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-configure bcmath --enable-bcmath \
    && docker-php-ext-configure intl --enable-intl \
    && docker-php-ext-configure pcntl --enable-pcntl \
    && docker-php-ext-configure mysqli --with-mysqli \
    && docker-php-ext-configure pdo_mysql --with-pdo-mysql \
    && docker-php-ext-configure pdo_pgsql --with-pgsql \
    && docker-php-ext-configure mbstring --enable-mbstring \
    && docker-php-ext-configure soap --enable-soap \
    && docker-php-ext-configure opcache --enable-opcache \
    && docker-php-ext-install -j$(nproc) \
        $PHP_EXTENSIONS \
    && pecl -d preferred_state=beta install xdebug \
    && pecl install \
        $PECL_EXT \
    && docker-php-ext-enable \
        $PECL_EXT
# PHP config
COPY conf.d/*.ini /usr/local/etc/php/conf.d/

ENV SSH_MODULES \
    bash git openssh rsync augeas shadow  curl wget

RUN apk add --no-cache \
        $SSH_MODULES

# SSH Sett up
RUN deluser $(getent passwd 33 | cut -d: -f1) \
    && delgroup $(getent group 33 | cut -d: -f1) 2>/dev/null || true \
    && mkdir -p ~root/.ssh /etc/authorized_keys \
    && chmod 700 ~root/.ssh/ \
    && augtool 'set /files/etc/ssh/sshd_config/AuthorizedKeysFile ".ssh/authorized_keys /etc/authorized_keys/%u"' \
    && echo -e "Port 22\n" >> /etc/ssh/sshd_config \
    && cp -a /etc/ssh /etc/ssh.cache \
    && rm -rf /var/cache/apk/*

COPY entry.sh /entry.sh

# Clean up
RUN apk del .build-deps \
    && rm -rf /tmp/* \
    && rm -rf /app \
    && mkdir /app

# Possible values for ext-name:
# bcmath bz2 calendar ctype curl dba dom enchant exif fileinfo filter ftp gd gettext gmp hash iconv imap interbase intl
# json ldap mbstring mcrypt mssql mysql mysqli oci8 odbc opcache pcntl pdo pdo_dblib pdo_firebird pdo_mysql pdo_oci
# pdo_odbc pdo_pgsql pdo_sqlite pgsql phar posix pspell readline recode reflection session shmop simplexml snmp soap
# sockets spl standard sybase_ct sysvmsg sysvsem sysvshm tidy tokenizer wddx xml xmlreader xmlrpc xmlwriter xsl zip

ENTRYPOINT ["/entry.sh"]
CMD ["/usr/sbin/sshd", "-D", "-e", "-f", "/etc/ssh/sshd_config"]
