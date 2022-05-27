# Docker for Laravel on Fly.io

Docker image build for use with Laravel 9.x applications on Fly.io. The primary goals are to both be a jumping off point for down-level images
as well as an example.

This image utilizes [Laravel Sail](https://github.com/laravel/sail) as a baseline, most notably as a mirror of the `/runtimes` directory.
We're also taking plenty of guidance from [Greg Sanderson's Fly Hello Laravel example](https://github.com/gregmsanderson/fly-hello-laravel).

As a general rule to speed up deploying applications, it seems better to bake in any OS level `apt` or `composer` package, within reason.
The largest bottleneck tends to be the network it runs on. Digital Ocean has a local cache of package managers but I'm not sure if Fly.io does.

This seems to make sense when you think that the docker image has to be downloaded at a certain compressed size then further packages have to travel
over the network.

### Table of Contents

* [Development](#development)
* [Official PHP image links](#official-php-image-links)

## Development

To support a new version of PHP:

1. Create a new top level directory, i.e. `8.1`.
2. Replicate the prior `Dockerfile`, essentially `cp 8.0/Dockerfile 8.1`.
3. Change the line `FROM php:8.0-fpm` to `FROM php:8.1-fpm`.
   1. This assumes the internal build tags don't change. Fortunately this tag is rather stable.
4. Build your new image to test: `docker build ./8.1 -t w0rd-driven/docker-laravel-deployer:8.1`.
5. Troubleshoot any breaking changes.
   1. Occasionally you may have to track a deviation further up the stack.
      1. Since we're running `php:8.1-fpm` you should inspect [the list of supported tags](https://github.com/docker-library/docs/blob/master/php/README.md#supported-tags-and-respective-dockerfile-links) to see what that image is built from.
   2. In the case of 7.3, `libzip` is no longer included so it needs to be added explicitly.
      1. 7.2 also removed extensions so this is the most common type of problem you'll run into.
   3. Remove any previously cached images via the rmi command: `docker rmi w0rd-driven/docker-laravel-deployer:8.1`.
   4. Remove any unused images `docker image prune --all`.

## Official PHP image

   1. [Docker Hub for official PHP image](https://hub.docker.com/_/php).
   2. [Github repository for official PHP image](https://github.com/docker-library/php).
