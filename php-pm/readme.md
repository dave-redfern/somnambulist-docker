# PHP Alpine for php-pm

An Alpine 3.9 based container geared for running [php-pm](https://github.com/php-pm/php-pm) that includes:

 * composer
 * ppm.phar (as ppm, from https://github.com/dave-redfern/somnambulist-phppm-phar)

PHP is installed with:

 * pcntl / posix and the memory extensions
 * igbinary
 * opcache / apcu
 * xml libs
 * curl
 
And a few others.

Note:

 * intl has not been included to reduce the base image size
 * only sqlite has been loaded, add MySQL / Postgres if you need them
 
The base image is around 50MB due to git and ncurses-terminfo being required by several packages.

## Intended Usage

Import from this image and add additional setup steps to build your app. For example:

```dockerfile
FROM somnambulist/docker:ppm

RUN apk --update add ca-certificates \
    && apk update \
    && apk upgrade \
    && apk --no-cache add -U \
    php7-pdo-pgsql \
    php7-intl \
    && rm -rf /var/cache/apk/* /tmp/*

WORKDIR /app

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod 755 /docker-entrypoint.sh

COPY composer.* ./
COPY ppm.* ./
COPY .env* ./

RUN composer install --no-suggest --no-scripts --quiet --optimize-autoloader

# copy all the files to the /app folder, use with a .dockerignore
COPY . .

# this should be the same port as defined in the ppm.json file
EXPOSE 8080

# certain settings could be overridden such as the ip / workers
#CMD [ "/docker-entrypoint.sh", "start", "--workers=2", "--cgi-path=/usr/bin/php-cgi", "--host=0.0.0.0" ]
CMD [ "/docker-entrypoint.sh", "start" ]
```

Where the `docker-entrypoint.sh` could be something like:

```bash
#!/usr/bin/env bash

set -e
cd /app

cmd="$@"

[[ -d "/app/var" ]] || mkdir -m 0777 "/app/var"
[[ -d "/app/var/cache" ]] || mkdir -m 0777 "/app/var/cache"
[[ -d "/app/var/logs" ]] || mkdir -m 0777 "/app/var/logs"
[[ -d "/app/var/run" ]] || mkdir -m 0777 "/app/var/run"
[[ -d "/app/var/run/ppm" ]] || mkdir -m 0777 "/app/var/run/ppm"
[[ -d "/app/var/tmp" ]] || mkdir -m 0777 "/app/var/tmp"

# run ppm, start should receive arguments from Dockerfile
/usr/bin/ppm $cmd
```

A `.dockerignore` should be setup to prevent copying in git and vendor files:

```
.idea
*.md
.git
.dockerignore
vendor
var
docker-compose*
```

Finally a `ppm.dist.json` should be added to the project that can be copied / modified
to the ppm.json by the docker-entrypoint if needed:

```json
{
    "bridge": "HttpKernel",
    "host": "0.0.0.0",
    "port": 8080,
    "workers": 2,
    "app-env": "docker",
    "debug": 0,
    "logging": 0,
    "bootstrap": "PHPPM\\Bootstraps\\Symfony4",
    "max-requests": 500,
    "max-execution-time": 60,
    "populate-server-var": true,
    "socket-path": "var\/run\/ppm\/",
    "pidfile": "var\/run\/ppm\/ppm.pid",
    "cgi-path": "\/usr\/bin\/php-cgi"
}
```

__Note:__ The Symfony4 bootstrap mentioned here is an extension provided by the compiled phar.
It replaces the standard Symfony bootstrap with one that can do kernel detection and supports
multiple environment files.

__Note:__ while php-pm can be used to serve full-stack applications, it works much better for
APIs that do not have to deal with assets and sessions.
