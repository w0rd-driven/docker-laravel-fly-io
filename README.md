# Docker for Laravel on Fly.io

Run Laravel 9.x applications on Fly.io. The primary goals are to be a solid jumping off point as well as an attempt at sane defaults for your application images.

We heavily leverage [Laravel Sail](https://github.com/laravel/sail) that facilitates local development containers but take them a step further to deploy for production workloads. Laravel Sail doesn't include Nginx, something the documentation highly encourages

We're also taking plenty of guidance from [Greg Sanderson's Fly Hello Laravel example](https://github.com/gregmsanderson/fly-hello-laravel).

As a general rule to speed up deploying applications, it seems better to bake in any OS level `apt` or `composer` package, within reason.
The largest bottleneck tends to be the network it runs on. Digital Ocean has a local cache of package managers but I'm not sure if Fly.io does.

This seems to make sense when you think that the docker image has to be downloaded at a certain compressed size then further packages have to travel
over the network.

### Table of Contents

* [This Has Been Abandoned](#this-has-been-abandoned)
    * [To Be Completed](#to-be-completed)
    * [Wishlist for Fly.io](#wishlist-for-flyio)
* [Features](#features)
* [Usage](#usage)
* [Configuration](#configuration)
    * [Nginx](#nginx)
    * [PHP](#php)
    * [Supervisor](#supervisor)
* [Development](#development)
* [Resources](#resources)

## This Has Been Abandoned

I had known Fly.io had been looking for [Laravel Specialists](https://fly.io/blog/fly-io-is-hiring-laravel-specialists/) and once Chris Fidao announced he joined, that things could quickly move in another direction.
[Full Stack Laravel](https://fly.io/laravel-bytes/full-stack-laravel/) shows us how easy it is to now configure a new application.
Compare that to the discussion [here](https://community.fly.io/t/using-laravel-on-fly-io/5037), the post that started me down this rabbit hole.

I've chosen to abandon this project largely because this solidified my desire to fully move away from PHP into Elixir.
When I read the other posts in [Laravel Bytes](https://fly.io/laravel-bytes/), I could no longer see myself as one of the authors.
It's funny how once something goes from imagination to seeing a concrete example, our brain quickly says either "Yep!" or "Nope!"
I may return to finish at some point but I want to be realistic and say that may never happen.

### To Be Completed

1. [This file is the most complete version](8.1/Dockerfile).
    1. My plan was to start with 8.1 and work backwards.
2. Remove the various Dockerfile.alpine versions.
    1. These were a holdover of my [previous Docker project](https://github.com/w0rd-driven/docker-laravel-deployer) spanning versions 7.4 to 8.1.
    2. I kept these around in the event the various OS packages or configuration would be useful. It may make sense to try to configure something from the PHP source instead of Ubuntu as done with Sail.
    3. Heavily used by Gitlab CI jobs at my employer, The Rethinkgroup, Inc.
3. Nginx is installed but nothing is configured.
    1. There's an existing command commented out to capture the default files as a backup. I always recommend this. It's more work to rebuild the image than to have files sitting around I may never use.
    2. My intention was to look at how Laravel Forge and the documentation suggests to run.
    3. Laravel Sail doesn't use Nginx so there's no guidance there to pull from.
4. I hadn't tested any of this in production yet but the image builds.
    1. There may be minor adjustments needed to the `supervisor` commands and arguments but I don't believe so. They seem pretty stable given what I've used with Laravel Forge.

### Wishlist for Fly.io

I've yet to compare this to the steps outlined in [Full Stack Laravel](https://fly.io/laravel-bytes/full-stack-laravel/) so these may be supported already.

1. In [queue workers](https://fly.io/docs/getting-started/laravel/#queue-workers).
    1. There's one large `docker/supervisor.conf` instead of one file per configuration.
    2. The directive `directory=/var/www/html/` is not used.
        1. The command becomes `php artisan` instead of always needing the path.
    3. I've been bit by just running `php artisan` without understanding which version that is.
        1. I prefer using `php8.1 artisan` to be explicit.
        2. Docker images generally don't have their PHP versions updated by devops.
2. [PHP v8.0 is used by default](https://fly.io/docs/getting-started/laravel/#php-version).
    1. This is honestly fine as what changes between PHP versions is usually negligible. That's not always the case though.
3. Inspiration from Sails
    1. A command that changes all of the settings in [the PHP section](https://fly.io/docs/getting-started/laravel/#php-version).
    2. A command to enable or change the `cron` and `queue` additions.
    3. Our version `8.1/etc/supervisor/conf.d/laravel-schedule.conf` runs a specific schedule instead of using `docker/cron`.
        1. This may or may not be an improvement, it's more personal preference.
    4. The configuration for the Redis section of [Full Stack Laravel](https://fly.io/laravel-bytes/full-stack-laravel/) would be great as a command.
        1. Something like `curl -s "https://laravel.build/example-app?with=mysql,redis" | bash` as suggested [here](https://laravel.com/docs/9.x/installation#choosing-your-sail-services).
        2. It'd be nice to have a `fly configure redis` or `php` that handled this for you.
        3. Editing the `fly.toml` should likely be an escape hatch if they hope to capture more Laravel Forge users. This is mostly how things are today but it could be taken a little bit further.

## Features

1. Sail images have one long `apt-get` command, we split them up into groupings.
    1. Con: This is annoying as changes upstream will require further understanding.
    2. Con: One long command produces a much smaller Docker image.
    3. Pro: Breaking apart logical sections makes it easier to understand.
    4. Pro: Docker caches each RUN line as a layer, if we're only tweaking Nginx none of the other sections get rebuilt.
    5. An example `sail/app` is `742.46 MB` versus `817.8 MB` but it's important to remember Nginx isn't included.
2. Supervisor files are split by need.
    1. Pro: We can disable or enable by moving the files in and out of the configuration directories.
    2. Pro: It's easy to find what to change vs a single larger file.
    3. Con: It may not be easier to determine what to change by file name alone.
3. Leveraging Laravel Sail
    1. Sail uses Ubuntu as a base over Alpine Linux.
        1. Pro: Users may be more used to working with Ubuntu.
        2. Con: Images are larger but not by a huge amount.
    2. Sail's `start-container` is a solid Docker entry point.
    3. Using Sail for personal and production development should make the transition far easier.

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
    * If any of these optional files are to be utilized, `group.conf` needs to be adjusted to start the new process.
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

    1. [Full Stack Laravel](https://fly.io/laravel-bytes/full-stack-laravel/)
    2. [Fly Hello Laravel](https://github.com/gregmsanderson/fly-hello-laravel)
    3. [Using Laravel on Fly.io](https://community.fly.io/t/using-laravel-on-fly-io/5037)
    4. Laravel Sail
        1. [Laravel Sail Repository](https://github.com/laravel/sail)
        2. [Laravel Sail Docker Environments](https://github.com/laravel/sail/tree/1.x/runtimes)
    5. [Docker in Development](https://serversforhackers.com/s/docker-in-development)
    6. Docker
        1. [Docker Hub for official PHP image](https://hub.docker.com/_/php).
        2. [Github repository for official PHP image](https://github.com/docker-library/php).
