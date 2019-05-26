#
# Dockerfile for PHP 7.2 specifically for background app processing
#
# Bundles composer and php-pm phar archives
#

FROM alpine:3.9
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /tmp

RUN apk --update add ca-certificates \
    && apk update \
    && apk upgrade \
    && apk --no-cache add -U \
    # Packages
    bash \
    curl \
    git \
    openssl \
    tini \
    unzip \
    wget \
    php7 \
    php7-common \
    php7-bcmath \
    php7-bz2 \
    php7-calendar \
    php7-cgi \
    php7-ctype \
    php7-curl \
    php7-dom \
    php7-ftp \
    php7-json \
    php7-mbstring \
    php7-opcache \
    php7-openssl \
    php7-pcntl \
    php7-pdo \
    php7-pdo_sqlite \
    php7-pecl-apcu \
    php7-pecl-event \
    php7-pecl-igbinary \
    php7-pecl-redis \
    php7-phar \
    php7-posix \
    php7-session \
    php7-shmop \
    php7-simplexml \
    php7-sodium \
    php7-sqlite3 \
    php7-sysvshm \
    php7-sysvshm \
    php7-sysvshm \
    php7-tokenizer \
    php7-xml \
    php7-xmlreader \
    php7-xmlwriter \
    php7-xsl \
    php7-zip \
    php7-zlib \
	# Clean up
    && rm -rf /var/cache/apk/* /tmp/*

# setup composer -- taken from the composer docker installer code
RUN curl --silent --fail --location --retry 3 --output /tmp/installer.php --url https://raw.githubusercontent.com/composer/getcomposer.org/cb19f2aa3aeaa2006c0cd69a7ef011eb31463067/web/installer \
 && php -r " \
    \$signature = '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5'; \
    \$hash = hash('sha384', file_get_contents('/tmp/installer.php')); \
    if (!hash_equals(\$signature, \$hash)) { \
      unlink('/tmp/installer.php'); \
      echo 'Integrity check failed, installer is either corrupt or worse.' . PHP_EOL; \
      exit(1); \
    }" \
 && php /tmp/installer.php --no-ansi --install-dir=/usr/bin --filename=composer \
 && composer --ansi --version --no-interaction \
 && rm -f /tmp/installer.php \
 && find /tmp -type d -exec chmod -v 1777 {} +

# setup php-pm -- shamelessly borrowed from the above composer code :)
RUN curl --silent --fail --location --retry 3 --output /tmp/ppm.phar --url https://github.com/dave-redfern/somnambulist-phppm-phar/releases/download/2.0.1/ppm.phar \
  && php -r " \
  \$signature = '54064b81d8390c60d1fa1fcf6964974728c176c3ec7a587a192384fd379f2668ed41ba047c7b6b6e15a073f3548577b9'; \
  \$hash = hash_file('sha384', '/tmp/ppm.phar'); \
  if (!hash_equals(\$signature, \$hash)) { \
    unlink('/tmp/ppm.phar'); \
    echo 'Integrity check failed, the ppm.phar archive is either corrupt or worse.' . PHP_EOL; \
    exit(1); \
  }" \
  && mv /tmp/ppm.phar /usr/bin/ppm.phar \
  && ln -s /usr/bin/ppm.phar /usr/bin/ppm \
  && chmod 755 /usr/bin/ppm \
  && ppm --ansi --version --no-interaction

ENTRYPOINT ["/sbin/tini", "--"]
