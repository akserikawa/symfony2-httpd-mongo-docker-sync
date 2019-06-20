FROM php:5.6.40-fpm

ADD php.ini /usr/local/etc/php/conf.d/php.ini

RUN apt-get update && apt-get upgrade -y \
    && apt-get install -y \
    g++ \
    libbz2-dev \
    libc-client-dev \
    libcurl4-gnutls-dev \
    libedit-dev \
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libkrb5-dev \
    libldap2-dev \
    libmagickwand-dev \
    libmcrypt-dev \
    libmemcached-dev \
    libpq-dev \
    libsqlite3-dev \
    libssh2-1-dev \
    libssh2-1 \
    libssl-dev \
    libreadline-dev \
    libxslt1-dev \
    memcached \
    wget \
    unzip \
    zlib1g-dev \
    && docker-php-ext-configure gd \
    --with-freetype-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install -j$(nproc) imap \
    && docker-php-ext-configure intl \
    && docker-php-ext-install -j$(nproc) intl \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
    && docker-php-ext-install ldap \
    && docker-php-ext-install -j$(nproc) \
    bcmath \
    bz2 \
    calendar \
    exif \
    gettext \
    mbstring \
    mcrypt \
    mysqli \
    opcache \
    pdo_mysql \
    pdo_pgsql \
    pgsql \
    soap \
    sockets \
    xmlrpc \
    xsl \
    zip \
    # && pecl install xdebug-2.5.5 && docker-php-ext-enable xdebug \
    && pecl install memcached-2.2.0 && docker-php-ext-enable memcached \
    && pecl install mongo-1.6.16 && docker-php-ext-enable mongo \
    && pecl install redis && docker-php-ext-enable redis \
    && pecl install ssh2 && docker-php-ext-enable ssh2 \
    && yes '' | pecl install imagick && docker-php-ext-enable imagick \
    && docker-php-source delete \
    && apt-get remove -y g++ wget \
    && apt-get autoremove --purge -y && apt-get autoclean -y && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* /var/tmp/*

# # Install composer
# RUN curl -o /tmp/composer-setup.php https://getcomposer.org/installer \
#     && curl -o /tmp/composer-setup.sig https://composer.github.io/installer.sig \
#     # Make sure we're installing what we think we're installing!
#     && php -r "if (hash('SHA384', file_get_contents('/tmp/composer-setup.php')) !== trim(file_get_contents('/tmp/composer-setup.sig'))) { unlink('/tmp/composer-setup.php'); echo 'Invalid installer' . PHP_EOL; exit(1); }" \
#     && php /tmp/composer-setup.php --no-ansi --install-dir=/usr/local/bin --filename=composer --snapshot \
#     && rm -f /tmp/composer-setup.*

# mssql stuff
RUN apt-get update ;\
    apt-get install -y --no-install-recommends \
    freetds-dev freetds-bin freetds-common libdbd-freetds libsybdb5 libqt4-sql-tds libqt5sql5-tds ;\
    ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/libsybdb.so ;\
    ln -s /usr/lib/x86_64-linux-gnu/libsybdb.a /usr/lib/libsybdb.a ;\
    docker-php-ext-install mssql ;\
    docker-php-ext-enable mssql ;\
    docker-php-ext-configure mssql

RUN docker-php-ext-configure pdo_dblib --with-libdir=lib/x86_64-linux-gnu && \
    docker-php-ext-install pdo_dblib \
    && docker-php-ext-enable pdo_dblib 