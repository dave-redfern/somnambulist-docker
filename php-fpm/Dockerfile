#
# Dockerfile for PHP 7.2 specifically for web apps
#
# Bundles composer
#

FROM somnambulist/php-base:latest

RUN apk --update add ca-certificates \
    && apk update \
    && apk upgrade \
    && apk --no-cache add -U \
    # Packages
    php7-fpm \
    # Clean up
    && rm -rf /var/cache/apk/* /tmp/*
