# PHP Alpine for php-fpm

An Alpine 3.9 based container geared for running more standard web apps e.g. Symfony 4+, that includes:

 * composer

PHP is installed with:

 * igbinary
 * opcache / apcu
 * xml libs
 * curl
 * iconv, mbstring, gettext, intl
 
And a few others.

Note:

 * only sqlite has been loaded, add MySQL / Postgres if you need them
 
## Intended Usage

Import from this image and add additional setup steps to build your app. For example:

```dockerfile
FROM somnambulist/php-fpm-alpine

RUN apk --update add ca-certificates \
    && apk update \
    && apk upgrade \
    && apk --no-cache add -U \
    php7-pdo-pgsql \
    && rm -rf /var/cache/apk/* /tmp/*

WORKDIR /app

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod 755 /docker-entrypoint.sh

# copy over the fpm config
COPY config/docker/dev/php-fpm/php-fpm.conf /etc/php-fpm.conf
COPY config/docker/dev/php-fpm/www.conf /etc/php-fpm.d/www.conf

# copy the php config / overrides, using zz- so it is loaded last
COPY config/docker/dev/php-fpm/custom.ini /etc/php.d/zz-custom.ini

COPY config/docker/dev/docker-entrypoint.sh /docker-entrypoint.sh

COPY composer.* ./
COPY .env* ./

RUN chmod 755 /docker-entrypoint.sh
RUN composer install --no-suggest --no-scripts --quiet --optimize-autoloader

# copy all the files to the /app folder, use with a .dockerignore
COPY . .

# this should be the same port as defined in the fpm.conf
EXPOSE 9000

CMD [ "/docker-entrypoint.sh" ]
```

And the `docker-entrpoint.sh` setups the environment before calling fpm:

```bash
#!/usr/bin/env bash

cd /app

[[ -d "/app/var" ]] || mkdir -m 0777 "/app/var"
[[ -d "/app/var/cache" ]] || mkdir -m 0777 "/app/var/cache"
[[ -d "/app/var/logs" ]] || mkdir -m 0777 "/app/var/logs"
[[ -d "/app/var/tmp" ]] || mkdir -m 0777 "/app/var/tmp"

/usr/sbin/php-fpm --nodaemonize
```

A `.dockerignore` should be setup to prevent copying in git and vendor files:

```
.idea
*.md
.git
.dockerignore
node_modules
vendor
var
docker-compose*
```

For node, setup another container as source, and use that to pre-build the node files or
better yet - setup it up in a web container and keep node_modules and the js / assets
out of the app code; though this may not work with EncoreBundle so adjust your build as
necessary.