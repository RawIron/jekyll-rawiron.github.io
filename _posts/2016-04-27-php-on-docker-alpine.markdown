---
layout: post
title:  "A small PHP docker image"
date:   2016-04-27 14:39:12 +0200
categories: devops docker
---

Docker was on my list of tools that I wanted to play with.
At work *Vagrant* has been the major tool for creating my local dev environment for some time now.
The major goal was to get something up and running and, while at it, dig around a bit.
I found a good read to get started was [an introduction on Linux Containers][linux-containers-intro].

The fastest track to get a Docker container with PHP is to use an image of one of the common Linux distributions like Ubuntu, RedHat etc. and add extensions and packages as required.
The available Docker images can be explored on [Docker Hub][docker-images].

Let's create a **small** Docker image.
So why a small image? - Three main reasons as far as I can tell:

1. the infrastructure is large. Meaning many developers, images, machines, etc.
2. a hard time limit on how long it can take to spin off a new machine with containers.
3. containers should be ephemeral. A bit like the Zen of Docker.

In a small company the few developers probably use a small amount of different images.
Also the images won't change too often.
It is worth checking if the Docker image size is of any concern.

When a company grows the number of images that float around and have to be kept somewhere is most likely growing fast.
Also new machines are deployed frequently, on production or test, and every time the docker images need to be transfered onto the new machines.
When the numbers get large then every improvement counts.

A docker image of Ubuntu is around *200MB* in size.
See [Brian's blog][brian-christner] that lists image sizes for selected Linux distributions.
In comparison the Java Runtime Environment is also around *200MB*.
Looks like the Linux distribution image is only one piece in the puzzle.

For the goal of creating a small Docker PHP image a good choice for the Linux distribution is the [Alpine Linux][alpine-linux] image.
There are two ways to get an Alpine Linux image with PHP on it:

1. start with the `alpine` image and add php.
2. start with the `php:alpine` image.


## alpine image
To add PHP to the `alpine` image Alpine's package manager `apk` is used.
Information about `apk` is on the [Alpine Wiki apk page][alpine-apk]
and the available *PHP* packages are in the [Alpine Package Repo][alpine-packages].

#### Build it
The `Dockerfile` to build the image is:
{% highlight dockerfile %}
FROM alpine

# install php
RUN apk update && apk add php
{% endhighlight %}

Create the image.
{% highlight bash %}
$ docker build -t "rawiron/alpine-php" .

Sending build context to Docker daemon 6.656 kB
Step 1 : FROM alpine
latest: Pulling from library/alpine
420890c9e918: Pull complete 
Digest: sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0
Status: Downloaded newer image for alpine:latest
 ---> d7a513a663c1
Step 2 : RUN apk update
 ---> Running in e5f6103291d1
fetch http://dl-cdn.alpinelinux.org/alpine/v3.3/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.3/community/x86_64/APKINDEX.tar.gz
v3.3.3-34-gf1fb2ef [http://dl-cdn.alpinelinux.org/alpine/v3.3/main]
v3.3.3-33-gd4025e6 [http://dl-cdn.alpinelinux.org/alpine/v3.3/community]
OK: 5858 distinct packages available
 ---> 4cb96c86bf87
Removing intermediate container e5f6103291d1
Step 3 : RUN apk add php
 ---> Running in d9d9befe9429
(1/9) Installing php-common (5.6.20-r0)
(2/9) Installing pcre (8.38-r0)
(3/9) Installing ncurses-terminfo-base (6.0-r6)
(4/9) Installing ncurses-terminfo (6.0-r6)
(5/9) Installing ncurses-libs (6.0-r6)
(6/9) Installing readline (6.3.008-r4)
(7/9) Installing libxml2 (2.9.3-r0)
(8/9) Installing php-cli (5.6.20-r0)
(9/9) Installing php (5.6.20-r0)
Executing busybox-1.24.1-r7.trigger
OK: 22 MiB in 20 packages
 ---> 010f27aa23c5
Removing intermediate container d9d9befe9429
Successfully built 010f27aa23c5
{% endhighlight %}

Print the size of the just built image.
{% highlight bash %}
{% raw %}
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Size}}"
{% endraw %}
IMAGE ID            REPOSITORY           SIZE
010f27aa23c5        rawiron/alpine-php   18.97 MB
d7a513a663c1        alpine               4.798 MB
{% endhighlight %}

Now create a container from the image.
{% highlight bash %}
$ docker run "rawiron/alpine-php" bash
{% endhighlight %}

And print the size of the container.
{% highlight bash %}
{% raw %}
$ docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Size}}"
{% endraw %}
CONTAINER ID        IMAGE                SIZE
a8bd6cb86e43        rawiron/alpine-php   0 B
{% endhighlight %}

Add a few PHP extensions.
{% highlight dockerfile %}
# add some php extensions
RUN apk update && apk add \
	php-json \
	php-phar \
	php-openssl
{% endhighlight %}

Print the size of the image with those extensions added.
{% highlight bash %}
{% raw %}
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Size}}"
{% endraw %}
IMAGE ID            REPOSITORY           SIZE
52d4a618af32        rawiron/alpine-php   19.54 MB
d7a513a663c1        alpine               4.798 MB
{% endhighlight %}

#### Composer
So far so good, but what if Composer is used to install the required packages.

{% highlight dockerfile %}
# install composer so it handles the php package installation
RUN echo "http://dl-3.alpinelinux.org/alpine/edge/testing" >>/etc/apk/repositories && \
    apk update && \
    apk add php-dom git composer
{% endhighlight %}

This size of the image doubled.
{% highlight bash %}
{% raw %}
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Size}}"
{% endraw %}
IMAGE ID            REPOSITORY                SIZE
f61eda9017b7        rawiron/alpine-composer   39.35 MB
13e1761bf172        alpine                    4.797 MB
{% endhighlight %}

#### Result
The Docker image is bare bones.
It only has the necessary *libs* to run basic PHP scripts.
With *Composer* added the image size is around *40MB*.


## php:alpine image
And now let's do this all over again with the *php:alpine* image.

#### Build it
The `Dockerfile` pulls the image.
{% highlight dockerfile %}
FROM php:5.6-alpine
{% endhighlight %}

Create the image.
{% highlight bash %}
$ docker build -t "rawiron/alpine-php" .

Sending build context to Docker daemon 6.656 kB
Step 1 : FROM php:5.6-alpine
5.6-alpine: Pulling from library/php
420890c9e918: Pull complete 
31a5a09e0543: Pull complete 
d22e6cb51c12: Pull complete 
42d1bf0f5e81: Pull complete 
a3ed95caeb02: Pull complete 
d5f11a6dba31: Pull complete 
8b95bb1992a6: Pull complete 
b8bfb3e680bd: Pull complete 
Digest: sha256:1bdad9120306566a711580f9749d03d8a657c9efcb1b43246cab7a2baffc4cc0
Status: Downloaded newer image for php:5.6-alpine
 ---> 503b35c9fea4
Successfully built 503b35c9fea4
{% endhighlight %}

> the *build* command created one image with two tags on it.
> the `docker rmi` command will fail when the list of image ids is generated as in `docker rmi $(docker images -q)`.
> see the [description and workaround of this issue][docker-rm-images]

And print the size of the just built image.
{% highlight bash %}
{% raw %}
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Size}}"
{% endraw %}
IMAGE ID            REPOSITORY           SIZE
503b35c9fea4        php                  334.6 MB
503b35c9fea4        rawiron/alpine-php   334.6 MB
{% endhighlight %}

#### Composer
After *Composer* was added to the image its size is close to *400MB*.

{% highlight bash %}
{% raw %}
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Size}}"
{% endraw %}
IMAGE ID            REPOSITORY                    SIZE
74ab7edb1116        rawiron/alpine-php-composer   363.6 MB
7bc0403e7fa9        php                           334.7 MB
{% endhighlight %}

#### Result
The image has a complete PHP install.
With Conmposer the image size is 9-times larger than the first build image.


[linux-containers-intro]: https://www.flockport.com/how-linux-containers-work/
[brian-christner]: https://www.brianchristner.io/docker-image-base-os-size-comparison/
[alpine-linux]: http://alpinelinux.org/
[alpine-packages]: https://pkgs.alpinelinux.org/packages
[alpine-apk]: http://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management
[docker-images]: https://hub.docker.com/explore/
[docker-rm-images]: https://github.com/docker/docker/issues/17304