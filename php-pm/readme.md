# Assorted Docker Containers

The Docker Hub repo is: somnambulist/docker. Each image is accessed via the tag.
The latest image version is without a version number, periodically versions will
be tagged.

## php-fpm (tag: fpm, fpm-X.Y.Z)

An alpine tailored php-fpm container

Includes:

 * composer
 * fpm

## php-pm (tag: ppm, ppm-X.Y.Z)

An alpine tailored php-pm container

Includes

 * composer
 * php-pm

Releases of the ppm container follow the php-pm release version.

## PHP Alpine for php-fpm

An Alpine 3.9 based container geared for running more standard web apps e.g. Symfony 4+, that includes:

 * composer

PHP is installed using the php-base image: [php-base](https://github.com/dave-redfern/docker-php-base) with:

 * fpm

Note:

 * only sqlite has been loaded, add MySQL / Postgres if you need them
 
### Intended Usage

Import from this image and add additional setup steps to build your app. For example:

```dockerfile
FROM somnambulist/docker:fpm

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
COPY config/docker/dev/php-fpm/php-fpm.conf /etc/php7/php-fpm.conf
COPY config/docker/dev/php-fpm/www.conf /etc/php7/php-fpm.d/www.conf

# copy the php config / overrides, using zz- so it is loaded last
COPY config/docker/dev/php-fpm/custom.ini /etc/php7/conf.d/zz-custom.ini

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



## PHP Alpine for php-pm

An Alpine 3.9 based container geared for running [php-pm](https://github.com/php-pm/php-pm) that includes:

 * composer
 * ppm.phar (as ppm, from https://github.com/dave-redfern/somnambulist-phppm-phar)

PHP is installed using the php-base image: [php-base](https://github.com/dave-redfern/docker-php-base) with:

 * cgi, pcntl, posix and the shm extensions.

Note:

 * only sqlite has been loaded, add MySQL / Postgres if you need them

### Intended Usage

Import from this image and add additional setup steps to build your app. For example:

```dockerfile
FROM somnambulist/docker:ppm

RUN apk --update add ca-certificates \
    && apk update \
    && apk upgrade \
    && apk --no-cache add -U \
    php7-pdo-pgsql \
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
