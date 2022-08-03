# Docker for Laravel on Fly.io

Run Laravel 9.x applications on Fly.io. The primary goals are to be a solid jumping off point as well as an attempt at sane defaults for your application images.

We heavily leverage [Laravel Sail](https://github.com/laravel/sail) that facilitates local development containers but take them a step further to deploy for production workloads. Laravel Sail doesn't include Nginx, something the documentation highly encourages

We're also taking plenty of guidance from [Greg Sanderson's Fly Hello Laravel example](https://github.com/gregmsanderson/fly-hello-laravel).

As a general rule to speed up deploying applications, it seems better to bake in any OS level `apt` or `composer` package, within reason.
The largest bottleneck tends to be the network it runs on. Digital Ocean has a local cache of package managers but I'm not sure if Fly.io does.

This seems to make sense when you think that the docker image has to be downloaded at a certain compressed size then further packages have to travel
over the network.

### Table of Contents

* [Usage](#usage)
* [Configuration](#configuration)
    * [Nginx](#nginx)
    * [PHP](#php)
    * [Supervisor](#supervisor)
* [Development](#development)
* [Resources](#resources)

## Usage

## Configuration

Nginx, php, supervisor, and any applicable application configuration is broken into grouping patterns to reduce changing lines in a larger
configuration file to replacing files entirely or just not copying them into the image.

### Nginx

### PHP

### Supervisor

This will likely be the biggest area of adjustment where not all files will be required and some will likely need to be adjusted further in the case
of manually configuring queue workers over running something like Laravel Horizon.

* Required
    * `docker.conf` - Sets supervisor defaults to run inside Docker containers.
    * `group.conf` - Determines which configurations to execute in the foreground. This will likely be the file that needs to be altered the most.
    * `nginx.conf` - Nginx proxies our HTTP requests to php-fpm.
    * `php-fpm.conf` - The main workhorse for our Laravel application, responsible for processing the web requests from Nginx.
    * Optional
    * `laravel-horizon.conf` - Laravel Horizon requires Redis but manages all our queue workers under a single process. You would typically configure either Horizon or the queue workers manually.
    * `laravel-notifications.conf` - Laravel queue worker configured for the `notifications` queue instead of default.
    * `laravel-queue.conf` - Laravel queue worker configured for the `default` queue.
    * `laravel-schedule.conf` - The Laravel scheduler.

## Development

To support a new version of PHP:

1. Create a new top level directory, i.e. `8.1`.
2. Replicate the prior `Dockerfile`, essentially `cp 8.0/Dockerfile 8.1`.
3. Change the line `FROM php:8.0-fpm` to `FROM php:8.1-fpm`.
    1. This assumes the internal build tags don't change. Fortunately this tag is rather stable.
4. Build your new image to test: `docker build ./8.1 -t w0rd-driven/docker-laravel-fly-io:8.1`.
5. Troubleshoot any breaking changes.
    1. Occasionally you may have to track a deviation further up the stack.
        1. Since we're running `php:8.1-fpm` you should inspect [the list of supported tags](https://github.com/docker-library/docs/blob/master/php/README.md#supported-tags-and-respective-dockerfile-links) to see what that image is built from.
    2. In the case of 7.3, `libzip` is no longer included so it needs to be added explicitly.
        1. 7.2 also removed extensions so this is the most common type of problem you'll run into.
    3. Remove any previously cached images via the rmi command: `docker rmi w0rd-driven/docker-laravel-fly-io:8.1`.
    4. Remove any unused images `docker image prune --all`.

## Resources

    1. [Fly Hello Laravel](https://github.com/gregmsanderson/fly-hello-laravel)
    2. Laravel Sail
        1. [Laravel Sail Repository](https://github.com/laravel/sail)
        2. [Laravel Sail Docker Environments](https://github.com/laravel/sail/tree/1.x/runtimes)
    3. [Docker in Development](https://serversforhackers.com/s/docker-in-development)
    4. Docker
        1. [Docker Hub for official PHP image](https://hub.docker.com/_/php).
        2. [Github repository for official PHP image](https://github.com/docker-library/php).
