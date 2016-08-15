# OntoWiki Development Environment
This repository provides a development environment based on Docker.

## Prepare the Development Environment

Clone the repository into the root folder of you OntoWiki checkout:

    git clone "https://github.com/pfrischmuth/ontowiki-devenv.git" devenv

**Tip**: Use `devenv` for the folder name, since it is ignored within the OntoWiki project.

Check the settings of your config files:

- Refer to the `docker-compose.yml` file for passwords, ports, database names, etc.
- The values in the default configuration files should match the dev env settings.

If you want you can add something like the following to you `/etc/hosts` file:

    <IP-of-Docker-Host> ontowiki.local

This will make your development setup available via the URL `http://ontowiki.local`.

## Use the Development Environment

Make sure you are in the root directory of your devenv clone and execute the following command:

    docker-compose up -d

This will start up the following containers in the background:

- ontowiki-devenv-nginx
- ontowiki-devenv-phpserver
- ontowiki-devenv-mysql-test
- ontowiki-devenv-mysql-dev
- ontowiki-devenv-virtuoso-test
- ontowiki-devenv-virtuoso-dev


If you want to inspect the logs of one of the containers execute for example:

    docker logs -f ontowiki-devenv-nginx

If you want to stop all containers enter:

    docker-compose stop

If you want to also remove the containers execute the following command instead:

    docker-compose down

## Access OntoWiki in the Browser

    http://<IP-of-Docker-Host>
    # or if configured in /etc/hosts
    http://ontowiki.local

## Run Tests

### Unit tests with bundled PHP

    make test-unit

or

    docker run --rm -v `pwd`/..:/var/www ontowiki-devenv/phpserver /bin/sh -c 'cd /var/www && php vendor/bin/phpunit --testsuite "OntoWiki Unit Tests"'

### Unit tests with other PHP versions

**Tip:** Check [https://hub.docker.com/_/php/](https://hub.docker.com/_/php/) for supported PHP versions.

Examples:

    # PHP 5.5
    docker run --rm -v `pwd`/..:/var/www php:5.5-cli /bin/sh -c 'cd /var/www && php vendor/bin/phpunit --testsuite "OntoWiki Unit Tests"'

or

    # PHP 5.6
    docker run --rm -v `pwd`/..:/var/www php:5.6-cli /bin/sh -c 'cd /var/www && php vendor/bin/phpunit --testsuite "OntoWiki Unit Tests"'

or

    # PHP 7.0
    docker run --rm -v `pwd`/..:/var/www php:7.0-cli /bin/sh -c 'cd /var/www && php vendor/bin/phpunit --testsuite "OntoWiki Unit Tests"'

or

    # Latest PHP
    docker run --rm -v `pwd`/..:/var/www php:cli /bin/sh -c 'cd /var/www && php vendor/bin/phpunit --testsuite "OntoWiki Unit Tests"'

### Integration tests with Virtuoso

    make test-virtuoso

or

    docker run --rm -v `pwd`/..:/var/www -e EF_STORE_ADAPTER=virtuoso --net devenv_default ontowiki-devenv/phpserver /bin/sh -c 'cd /var/www && php vendor/bin/phpunit --testsuite "OntoWiki Virtuoso Integration Tests"'

### Integration tests with MySQL

    make test-mysql

or

    docker run --rm -v `pwd`/..:/var/www -e EF_STORE_ADAPTER=zenddb --net devenv_default ontowiki-devenv/phpserver /bin/sh -c 'cd /var/www && php vendor/bin/phpunit --testsuite "OntoWiki Virtuoso Integration Tests"'

### Integration tests with Stardog

    make test-stardog

or

    docker run --rm -v `pwd`/..:/var/www -e EF_STORE_ADAPTER=stardog --net devenv_default ontowiki-devenv/phpserver /bin/sh -c 'cd /var/www && php vendor/bin/phpunit --testsuite "OntoWiki Virtuoso Integration Tests"'

## Tips & Tricks

### Use `docker-machine` (e.g. on a Mac)

Create a new docker machine for OntoWiki development (optional):

    docker-machine create -d virtualbox ow-dev

Use the ow-dev machine in the current terminal session:

    eval $(docker-machine env ow-dev)

Retrieve the IP address of the docker machine:

    docker-machine ip ow-dev

## Manual Container Management (without `docker-compose`)

### Virtuoso

OntoWiki:

    docker run -d --name virtuoso-dev -v /data -e DBA_PASSWORD=owdev -e SPARQL_UPDATE=true -p 8890:8890 -p 1111:1111 tenforce/virtuoso

Tests:

    docker run -d --name virtuoso-test -v /data -e DBA_PASSWORD=owdev -e SPARQL_UPDATE=true -p 8891:8890 -p 1112:1111 tenforce/virtuoso

### MySQL

OntoWiki:

    docker run -d --name mysql-dev -v /var/lib/mysql -e MYSQL_RANDOM_ROOT_PASSWORD=1 -e MYSQL_USER=php -e MYSQL_PASSWORD=owdev -e MYSQL_DATABASE=ontowiki-dev -p 3306:3306 mysql

Tests:

    docker run -d --name mysql-test -v /var/lib/mysql -e MYSQL_RANDOM_ROOT_PASSWORD=1 -e MYSQL_USER=php -e MYSQL_PASSWORD=owdev -e MYSQL_DATABASE=ow_TEST -p 3307:3306 mysql

### PHP

Build the custom image (OntoWiki needs some PHP extensions):

    docker build -t ow_php docker/php

Run a container based on that image:

    docker run -d --name phpserver --link virtuoso-dev:virtuoso-dev --link mysql-dev:mysql-dev -v `pwd`/..:/var/www -p 9000:9000 ow_php

### Nginx

Build the custom image (provides nginx configuration for OntoWiki):

    docker build -t ow_nginx docker/nginx

Run a container based on that image:

    docker run -d --name nginx --link phpserver:phpserver -v `pwd`/..:/var/www -p 80:80 ow_nginx
