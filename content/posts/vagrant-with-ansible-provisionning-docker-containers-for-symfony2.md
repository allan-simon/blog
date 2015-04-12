+++
date = "2015-04-12T04:14:20+08:00"
draft = true
title = "vagrant with ansible provisionning docker containers for symfony2"

+++

In this article we're going to cover:

  * How to use ansible+docker+vagrant to create a one size for all environment for Symfony2 + Nginx + Postgresql + PHP-fpm
  * How to transition an existing application to fit in that new environment

This article take the assumption that you have already heard about all the technologies above without being too familiar with them.


As an example will take the existing Oauth2 server I have created: [Oauth2Symfony2](https://github.com/allan-simon/oauth2-symfony2-vagrant-fosuserbundle)

### Introduction

Recently I've been reading a lot about Docker, Vagrant and Ansible.
At my company we're already using vagrant and ansible to create our
development machines, and as now the continous testing tool is 
using a docker container to run the test, the next step was logically to start
using docker everywhere, in order to have the same environment from the
developers machine to production.

My requirements were the following:

  * Get a model to split our each of our web projects into a set of 
docker containers, to increase reusability and reproductability
  * Use ansible for provisionning, for its powerful feature while yet
staying simple.
  * Be able to have one command, the same, to create my dev environment on my linux
laptop and to create staging / production etc.
  * Still be able to have a Vagrant for the developers on Mac/Windows but just
as a layer to provide them the linux necessary to have Docker (and ansible)
  * In case of Heisenbug, be able to nuke my environment and recreate quickly
in a way I'm sure to have something clean, while being sure not to have broken
my computer
  * To works fine with Symfony2 web applications


## Step 1, getting the Docker containers ready

For this first try, we're not going to push the logic to its maximun
and we will keep things simple we're going to have 2 Docker containers

  * Postgresql container hosting the database
  * Nginx+PHP-fpm container hosting the application code

We will put all our dockerfiles in a directory `DockerFiles` with
one subdirectory by container.

### Postgresql container 

The docker repository has already a very nice [official postgresql container](https://registry.hub.docker.com/_/postgres/)
it contains instructions on how to extend it, perfect

So in `DockerFiles/postgres` we now have

```
# vim:set ft=dockerfile:
FROM library/postgres
```

Here nothing funky, we're just saying our container's image will use
the official postgres image as a base. As we progress it will be completed with a script to create a database and a database user, but outside of this it currently perfectly fits our needs

### Nginx + PHP-fpm

The docker repository is full of container for PHP-fpm with Nginx but I didn't find any which met my needs:

  * Getting a recent version of PHP (5.5)
  * PHP with enough PHP-modules pre-instaled (php5-redis and php-posgresql)
  * Small image size (we're located in China and 300mo instead of 500mo can means an hour or two saved)


So I started from a the stock Debian Wheezy image, Debian and not ubuntu because that's on what the Postgres image is based, and it will not necessitate to redownload an other full distribution.

Here's the base docker file we're creating in `DockerFiles/Symfony2/`, comments have been added inside

```
# vim:set ft=dockerfile:
FROM debian:wheezy

# so that my colleagues know who to yell at
# when they will find a problem in it.
MAINTAINER SIMON Allan <simona@gobeta.com.cn>

# RUN is going to create a new layer, in order to avoid creating
#     dozen of layers, we chain the command with &&
#
# we're adding packages.dotdeb.org repository to get PHP5.5
# and activating the backport to get php5-redis
#
# once done we install the necessary package
# curl to later get composer)
# git to clone non-stable composer packages
# a huge list of php5 modules to cover future usage
#
# at the end we clean all temporary files so that when docker
# create the layer, it is as slim as possible
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E9C74FEEA2098A6E && \
    echo "deb http://packages.dotdeb.org/ wheezy-php55 all" > /etc/apt/sources.list.d/php.list && \
    echo "deb http://ftp.debian.org/debian wheezy-backports main contrib non-free" >> /etc/apt/sources.list.d/php.list && \
    apt-get update && \
    apt-get install -y \
        curl \
        git \
        nginx \
        php5-fpm \
        php5-cli \
        php5-xdebug \
        php5-imagick \
        php5-gd \
        php5-mongo \
        php5-curl \
        php5-mcrypt \
        php5-intl \
        php5-mysql \
        php5-sqlite \
        php5-redis \
        php5-pgsql \
    && rm -rf /var/lib/apt/lists/*

# we directly put composer in the image, so that
# even in case of China blocking the composer website, the image
# will still be usable, as this command may fails (thanks great firewall)
# we put it on a separate RUN, so that we don't need to run the previous
# commands again and again in case of failure.
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# our docker image will put this port as 'public', and then people
# using this image will be able to map it to port on their host machine
EXPOSE 80
EXPOSE 443

# a good practice in docker, as the container themselves can be run
# as demon, is to run the service inside it directly, so that the
# ouput of the service goes on STDOUT and can be gathered using
# standard tool (more on that in an other article)
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

```

### Building the Docker images using Ansible

For those who don't know [Ansible](http://docs.ansible.com/), it's a tool made to automate all the steps you do in order to setup an environment, be it creating files, installing packages, modifying configuration, making sure a service is started and an other stopped. It works also if the environment is made of several machines scattered over the world, as long as you can SSH to them.

Here the first goal is to make Ansible build these docker images. Latter we will improve it to start containers based on these images and linking them together.

Ansible is used by creating what they call a "playbook", a playbook is one or several Yaml files describing the tasks to accomplish or the state to reach. Common tasks like installing packages have pre-made modules to make their description short and easy-to-read.

Before anything make sure you have the latest version of ansible `pip install ansible --upgrade`, we assume from now that you have the versio `1.9.0.1`. Then at the root of your working directory create a file name `playbook.yml` and put that in it

```
---
- hosts: all
  sudo: yes
  tasks:
    - name: Install Docker-py
      pip: name=docker-py state=present

    - name: check or build image for postgres
      docker_image: path="./DockerFiles/postgres" name="allansimon/postgres-for-symfony" state=build

    - name: check or build image for symfony2
      docker_image: path="./DockerFiles/Symfony2" name="allansimon/symfony2" state=build


```

Here we have a very basic playbook that can be applied to every hosts and that will run every tasks using sudo. Then we describe one task (`name` is up to you) that will use `pip` to install `docker-py` if not already present. Docker-py will be needed for ansible to communicate with Docker.

Then we have two tasks that will build the Dockerfile in `path` and will associate it ot a name.

 **Note:** the `docker_image` is marked as deprecated in the Ansible, however the new `docker` module has nothing to build Dockerfiles...

 **Note 2:** on ubuntu 14.10 I got a bug with the `pip` installed by apt and I needed to add these tasks at the beginning 

```
  tasks:
    - name: remove pip if installed from apt (fix bug of pip in ubuntu 14.XX)
      apt: name=python-pip state=absent

    - name: Easy install (fix bug of pip in ubuntu 14.XX)
      easy_install: name=pip
    # then the other tasks

```

### Modifying our Postgres image to create a database for Symfony

Our postgres image is currently very basic, there's postgresql installed inside but no database or user created for our application. Let's fix that.According to the [documentation](https://registry.hub.docker.com/_/postgres/) you can add your custom script or SQL query to be launched at the container start by adding your script to the `/docker-entrypoint-initdb.d` directory. Let's do that, create a file `DockerFiles/postgres/make_db.sh`

with this content

```
#!/bin/bash
echo "******CREATING DOCKER DATABASE******"
gosu postgres postgres --single <<- EOSQL
    CREATE USER "$APP_DB_USER_NAME" WITH PASSWORD '$APP_DB_USER_PASSWORD';
    CREATE DATABASE "$APP_DB_NAME" WITH OWNER="$APP_DB_USER_NAME" ENCODING='UTF-8';
EOSQL
echo "******DOCKER DATABASE CREATED******"

```

We're going to see just after where the variables come from.

and now lets tell docker to add this script inside the image and to make it executable 

```
# vim:set ft=dockerfile:
FROM library/postgres

MAINTAINER SIMON Allan <simona@gobeta.com.cn>

ADD make_db.sh /docker-entrypoint-initdb.d/
RUN chmod +x /docker-entrypoint-initdb.d/make_db.sh 
```

Et voila! nothing more for the Dockerfile of postgres.

**/!\ One important remark, the file is added once, it means that if you latter modify the file, you need to rebuild the image. For that the 'easy' way I found is to add a blank line in the dockerfile to force it to rebuild, but I guess there's a better way.**


### Start a Postgresql container based on our image from Ansible.

Now that our image is ready, it's time to start it, add at the end of your `playbook.yml`

```

    # we define a new task
    - name: postgresql container
      # this task use the module docker to manage docker from ansible
      docker:
        # it's going to start a container based on our image
        image: allansimon/postgres-for-symfony
        # this specific container will be named app_database
        name: app_database
        # if the container does not exist it will be started, if already
        # started it will be restarted (it was useful while creating
        # the image
        state: restarted
        # we map our local /tmp/postgres to /var/lib/postgresql/data
        # in the container, this way the database is persisted
        volumes:
            - /tmp/postgres:/var/lib/postgresql/data
        # this precise the environment variable that will be given
        # to the container with their value
        env:
            # this one is needed by the base postgres container
            # to define a password to postgres user
            # the explanation about {{ }} notation is coming
            POSTGRES_PASSWORD: "{{ DB_PASSWORD }}"
            # here we have the variables used by our make_db.sh script
            APP_DB_USER_NAME: "{{ APP_DB_USER_NAME }}"
            APP_DB_USER_PASSWORD: "{{ APP_DB_USER_PASSWORD }}"
            APP_DB_NAME: "{{ APP_DB_NAME }}"


```

The `{{ }}` notation is for Ansible variables, there's several ways to declare, we're going to see two, that can be used at the same time, either by putting them directly in your `playbook.yml` or in a dedicated file. The advatange of the second is that this way you can add this file to your gitignore.

```
  vars:
    DB_PASSWORD: postgres
  vars_files:
    - external_vars.yml
```

and in `external_var.yml`

```
#for symfony
APP_DB_USER_NAME: symfony_db_user
APP_DB_USER_PASSWORD: symfony_db_password
APP_DB_NAME: symfony_db
```

### Start a Nginx+PHP-fpm container linked to the Postgresql one

Now that we have our postgres container started it's time to do the same with our container for Nginx and PHP-fpm it takes the same structure as the task for postgres, with just some little thing in more (explained in comments)

```
    - name: nginx and php-fpm
      docker:
        name: app_webserver
        image: allansimon/symfony2
        state: restarted
        # here we say that the container 'app_database' will be linked
        # to this container, with the local name app_database, i.e all
        # the env variables of app_database will be accessible from app_webserver
        # prefixed with `APP_DATABASE_ENV_` as well as the port exposed by
        # the container (so only the webserver will have access to the db)
        links:
            - "app_database:app_database"
        # Here as we need the webserver to be accessible from the outside
        # we map our laptop port 8088 to the port 80 of the container
        ports:
            - "8088:80"
        # it is possible to map as many directory as you need
        # the only restriction is that it must be an asbolute path
        # hence why we use a variable APP_DIR
        volumes:
            - "{{ APP_DIR }}:/var/www"
            - /tmp/nginx-logs:/var/logs/nginx
        env:
            # more on these variables sooon
            GITHUB_TOKEN : "{{ GITHUB_TOKEN  }}"
            APP_DB_USER_NAME: "{{ APP_DB_USER_NAME }}"
            APP_DB_USER_PASSWORD: "{{ APP_DB_USER_PASSWORD }}"
            APP_DB_NAME: "{{ APP_DB_NAME }}"
            APP_MAILER_HOST: "{{ APP_MAILER_HOST }}"
            APP_MAILER_USER: "{{ APP_MAILER_USER }}"
            APP_MAILER_PASSWORD: "{{ APP_MAILER_PASSWORD }}"
```

With that, our Symfony app will have all the information it needs to access to the database, and to be accessed from the outside


### Finalize the configuration of the Nginx+PHP-fpm image

Our two containers are now able to communicate but the same as we needed to create a database to make our postgres container useful, we're going to need to tweak a little our Nginx+PHP-fpm container in order to

  * have a virtualhost that sends file to PHP-fpm
  * get a timezone precised in our php.ini (otherwise symfony2 will refuse
to work)
  * have our container that run composer install when started
  * get nginx service started as well as php-fpm

so we're going to add this at the end of our `DockerFiles/Symfony2/Dockerfile`

```
# we put our self-defined vhost (see below for the content)
COPY config/vhost.conf /etc/nginx/sites-enabled/default

# we put our additional php configuration and we enable it
COPY config/php-extra.ini /etc/php5/mods-available/extra.ini
RUN php5enmod extra

# we copy our script and we make sure it is executable
COPY entrypoint.sh /root/entrypoint.sh
RUN chown root:root /root/entrypoint.sh 
RUN chmod +x /root/entrypoint.sh

# more on that in a latter article
VOLUME ["/var/www", "/var/log/nginx/"]

# when our container is started this script will be run
# (it was not in our postgres dockerfile because the base image already
# had it.
ENTRYPOINT ["/root/entrypoint.sh"]

```

now the configuration file themselves

#### DockerFiles/Symfony2/config/vhost.conf

This vhost is specifically for symfony2 applications
you maybe to adapt it if you plan to run other kind
of php websites.
```

server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    server_name localhost;

    root /var/www/web;
    index index.html index.htm index.php;

    location / {
        try_files $uri /app.php$is_args$args;
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/www;
    }

    location ~ ^/(app|app_dev|config)\.php(/|$) {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTPS off;
    }
}

```

#### DockerFiles/Symfony2/config/php-extra.ini

nothing fancy in it

```
date.timezone = UTC
```

#### DockerFiles/Symfony2/entrypoint.sh

```
#!/bin/bash

# for development machines
if [ $DEBUG ]; then
    echo "xdebug.remote_connect_back=On" >> /etc/php5/fpm/conf.d/20-xdebug.ini
    echo "xdebug.remote_enable=On" >> /etc/php5/fpm/conf.d/20-xdebug.ini
fi

cd /var/www

# if no specific command are precised when the container is started:
if [ -z "$1" ];
    then

    # if you're in china and that for some reason
    # the docker build has failed to download composer.phar...
    if ! which composer; then
        curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
    fi
    # variable provided by ansible when launching the docker container
    mkdir ~/.composer
    cat > ~/.composer/config.json <<EOS
{
    "config": {
        "github-oauth": { "github.com": "$GITHUB_TOKEN" }
    }
}
EOS
    # we make sure to start fresh
    rm app/config/parameters.yml
    composer install
    rm -rf app/cache/*
    php app/console assets:install --symlink web/
    php app/console c:c
    php app/console c:w
    # not pretty ...
    chmod -R 777 app/logs
    chmod -R 777 app/cache

    service php5-fpm restart
    nginx
else
    exec "$@"
fi
```

Several things:

  * We create a config.json for composer to put inside our github token to avoid composer install to hang because we're reached github anonymous limit.
  * We delete the parameters.yml so that we're sure it's recreated everytime, which is important in case we change the environment variables or the parameters.yml.dist
  * we clear and warmup the cache
  * it's a bit hackish , but we make sure the logs and cache directory are writeable by symfony. (I did like this because it seems the application is run with a user that only have a UID but no username, and searching on the internet, different people seems to have different UID)
  * then we start the php5-fpm service (in a perfect docker world, it should be on dedicated container so that we can easily its output)
  * we start nginx as a process in the foreground


To get your github token, you can take a look at my previous article on [how to generate one to use with composer.](http://allan-simon.github.io/blog/posts/solve-composer-github-api-limitation/)


Now our environment is fully ready we're only left with adapting our appliction

### Adapting our symfony application to fit in

Here's there's nothing much to change itself in the code, we simply need to tell Symfony2 to look in priority for the environment variables. The problem is Symfony2 itself looks for variable starting with `SYMFONY__` and the second problem is that the `parameters.yml` has priority over the environment variables...

To solve that there's a simple trick, (first make sure your parameters.yml is not versionned), composer.json has a package that can be used to generate your parameters.yml and it can be used to generate it based on environment variables

Make sure you have in your composer.json

```
"require": {
    ...

    "incenteev/composer-parameter-handler": "~2.0",
    ...
}
...
"scripts": {
    ...
    "post-install-cmd": [
        "Incenteev\\ParameterHandler\\ScriptHandler::buildParameters",
        ...
    ],
    "post-update-cmd": [
        "Incenteev\\ParameterHandler\\ScriptHandler::buildParameters",
        ...
    ]
},

```

Then in the `extra` section you can add `env-map` section to say to which environment variables to use (if present) to generate which parameter of your `parameters.yml`

In our case 

```
    "extra": {
        ...
        "incenteev-parameters": {
            "env-map" : {
                "database_host": "APP_DATABASE_PORT_5432_TCP_ADDR",
                "database_port": "APP_DATABASE_PORT_5432_TCP_PORT",

                "database_name": "APP_DB_NAME",
                "database_user": "APP_DB_USER_NAME",
                "database_password": "APP_DB_USER_PASSWORD",

                "mailer_host": "APP_MAILER_HOST",
                "mailer_user": "APP_MAILER_USER",
                "mailer_password": "APP_MAILER_PASSWORD"
            },
            "file": "app/config/parameters.yml"
        },
        ...
    }
```

The two first environment variables come from the variables given by docker to the `app_webserver` when linking with `app_database`

### Running everything 

Now that everything is done you can run your playbook using:

```
sudo  ansible-playbook playbook.yml --connection=local -i "[default] localhost," 
```
it tells to run the playbook on your local machine.

### Link it to vagrant
All of that works fine if you're on a Linux machine, so for your fellow Mac or Windows colleague you can create this `Vagrantfile`

```

def host_box_is_unixy?
  (RUBY_PLATFORM !~ /cygwin|mswin|mingw|bccwin|wince|emx/)
end


Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/trusty64"

    config.vm.network "forwarded_port", guest: 80, host: 8080
    config.vm.network "private_network", ip: "192.168.50.12"

    if host_box_is_unixy?
        config.vm.synced_folder "./", "/vagrant", type: "nfs"
    else
        config.vm.synced_folder "./", "/vagrant", type: "smb", mount_options: ['ip=192.168.50.1'] #host side of :private_network
    end


    config.vm.provision :shell, :inline => <<-END
        set -e
        if ! which ansible-playbook ; then
            sudo sh -c "wget -qO- https://get.docker.io/gpg | apt-key add -"
            sudo sh -c "
                echo deb http://get.docker.io/ubuntu docker main > \
                /etc/apt/sources.list.d/docker.list
            "
            sudo apt-get update
            sudo apt-get -y install \
                lxc-docker \
                python-software-properties \
                python-pip
            sudo pip install ansible
        fi
        cd /vagrant
        sudo  ansible-playbook playbook.yml --connection=local -i "[default] localhost," 
    END
end
```

it is not treated in this article but for Symfony2 performance reason on Vagrant, you may want to put the cache and logs folder outside of the share folder.

### Useful commands 

### Run a command in a given container

for example you want to run phpunit

```
sudo docker exec -it app_webserver   /var/www/bin/phpunit -c  /var/www/app
```

or if you want to 'ssh' into the machine 

```
sudo docker exec -it app_webserver bash
```

### Get the logs

```
sudo docker logs -f app_webserver   
```

### Delete an image

Let's say it's 3AM you're trying desperately to modify the Dockerfile of the database but nothing change, so you want to nuke everything and start fresh

First stop and remove the container using

```
sudo docker stop app_database2
sudo docker rm app_database2

```

then delete the image itself

```
sudo docker rmi allansimon/postgres-for-symfony
```

### Further improvements

With that you should have a good starters to have your application running in dockers , while still providing an easy environment to deploy for your developers, but things can be of course furthered improve (I will try to cover them in other articles)


  * Integrate it smoothly with your continous-testing system (for example gitlab-ci)
  * Split the containers in smaller ones (for example nginx and php-fpm in separate ones)
  * Put the data in Docker volumes to easily backup your data
  * Have a procedure to easily rollout a new release that include database migrations (using the excellent DoctrineMigration) without stopping the service or breaking things
  * Use kubernetes to manage your running containers to restart them if one crash etc.
  * Make your application scalable by being able to pop-out more instance of the web application and getting it to work nice with a loadbalancer
  * Make your database scalable by using tools like `pgpool II`
  * Be able to have our containers over several physical machines

### Articles that helped me wrote this one and further reading

#### About the content of this article
  * [Ansible documentation on docker](http://docs.ansible.com/docker_module.html)
  * [Ansible documentation on docker images](http://docs.ansible.com/docker_image_module.html)
  * [How to build docker images with Ansible](http://patg.net/ansible,docker/2014/06/20/ansible-docker-image/)
  * [Other article on creating docker images with Ansible](http://victorlin.me/posts/2014/08/11/building-docker-image-with-ansible)
  * [Dockerizing Symfony Application](http://www.slideshare.net/denderello/dockerizing-symony-applications-symfony-live-berlin-2014)
  * [Building HA Application with Docker](http://www.slideshare.net/nevalla/building-high-availability-application-with-docker)
  *  

#### Further reading

  * [How to link docker containers accross host](http://devopscube.com/how-to-link-docker-containers-across-hosts-the-ambassador-pattern/)
  * [Using Apache Mesos to get a production ready environment](http://getprismatic.com/story/1428327711576?share=MzE2MzYy.MTQyODMyNzcxMTU3Ng.6TZneNzk4ocS2YDOFkjk5AM0r-Y)
  * [Docker swarm](http://www.slideshare.net/rajdeep/docker-swarm-introduction?next_slideshow=1)
  * [List of articles on Docker](http://www.scoop.it/t/docker-by-docker)
  * [Doing rolling update with ansible](http://docs.ansible.com/playbooks_delegation.html)
  * [How to scale a Laravel application with postgresql](https://www.digitalocean.com/community/tutorials/how-to-horizontally-scale-a-laravel-4-app-with-a-postgresql-database)
  * [Rock solide deployement of Symfony application](http://www.slideshare.net/pgodel/symfony-live-nyc-2014-rock-solid-deployment-of-symfony-apps)
