---
layout: post
title:  "Run PHP on Alpine Linux inside a Docker container"
date:   2016-04-27 14:39:12 +0200
categories: devops docker
---

The fastest track to get a Docker container with PHP is to use one of the common Linux distributions like Ubuntu, RedHat etc. and add extensions and packages as required.

Let's be ambitious and create a **small** Docker container.
Selected image sizes are listed in [Brian's blog][brian-christner].

So a good choice to start with is the [Alpine Linux][alpine-linux] image.
Two ways to get this done:

1. start with the `alpine` image and add php.
2. start with the `php:alpine` image.

Available Docker images can be explored on [Docker Hub][docker-images].

### alpine image
The package manager of *Alpine* is `apk`.
Add the required *php* packages from the [Alpine Package Repo][alpine-packages].
More info about `apk` is on the [Alpine Wiki apk page][alpine-apk].

### php:alpine image
A `Dockerfile` that does this:

{% highlight dockerfile %}
FROM php:5.6-alpine

#---> SYSTEM <---

# install composer so it handles the php package installation
RUN apk update && \
    apk add curl git

RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer


#---> DEPLOY <---

# create the app directory
RUN mkdir -p /var/www/app

# php packages
COPY composer.json /var/www/app/composer.json
RUN cd /var/www/app && \
    composer install

# configure phpunit
COPY phpunit.xml /var/www/app/phpunit.xml

# deploy source

# run tests
RUN cd /var/www/app && \
    phpunit

{% endhighlight %}

[brian-christner]: https://www.brianchristner.io/docker-image-base-os-size-comparison/
[alpine-linux]: http://alpinelinux.org/
[alpine-packages]: https://pkgs.alpinelinux.org/packages
[alpine-apk]: http://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management
[docker-images]: https://hub.docker.com/explore/
