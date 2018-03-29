+++
date = "2015-03-29T15:05:36+08:00"
draft = true
title = "gitlab ci with docker runner"

+++

In my company we're using [Gitlab](https://gitlab.com) for managing our code
and doing code review. For the last months we've been standardizing our stack
on Symfony/Postgresql/php-fpm. Recently we've started automatizing our tests
using phpunit. The next step was of course to have a start of continuous 
integration and to be able to run our tests at each Merge request.

In this article we're going to view

  * How to install gitlab-ci on a separate server than your gitlab instance
  * How to put gitlab-ci behind your company frontal server
  * How to make gitlab-ci run your tests in a Docker container

<!--more-->

## Gitlab-CI in a separate instance

To do continous-integration you got plenty of choice, the most popular one being
[jenkinks](http://jenkins-ci.org). However it seems to be that it is a bit bloated
and has too much feature that we don't need for the moment.

On the other side gitlab comes with its own CI system (named Gitlab-ci) that has
the advantage to be damn simple for the moment and of course pretty much integrated
to gitlab (no need to create new accounts, build status directly in gitlab etc.)

In order to install it you can use the [ommibus package](https://about.gitlab.com/downloads/)
which will take care of installing everything from you

Then in order to have only gitlab-ci you can can follow the step [Running Gitlab on its own server](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/gitlab-ci/README.md)

## Gitlab-CI behind your company's frontal server

Our company as one public IP under which we host all our services.
We have then a frontal apache2 (but it could have been a nginx) which proxify
then the request depending of the hostname. In that case it does not make
sense for us to still have the Nginx that is installed with the omnibus package

So in order to deactivate it, in your `/etc/gitlab/gitlab.rb` in the Nginx section
you can add 

```
nginx['enable'] = false
ci_nginx['enable'] = false
```

and then 

```
sudo gitlab-ctl reconfigure 
```

then in your frontal you can put a vhost

```
<VirtualHost  *:80>
    ServerName gitlab-ci.example.com

    ProxyRequests off
    ProxyPreserveHost on
    ProxyPass / http://LAN_IP_OF_YOUR_GITLAB_CI:8181/
    ProxyPassReverse / http://LAN_IP_OF_YOUR_GITLAB_CI:8181/
</VirtualHost>
```
(by default gitlab-ci's unicorn listen on port 8181)

## Tell Gitlab-ci to run your test into a docker container

Gitlab-ci use what they call a "Runner" to run your test, it's more
or less a spare machine in which Gitlab-ci will clone your project 
and run a bash script you've provided in the Gitlab-ci admin panel

The goal of this article is not to do a deep introduction to [Docker](https://www.docker.com/)
but basically it's a powerful technology that permits you to run "containers".
You can consider a container as a virtual machine without the virtual part, and restricted
to Linux. (more or less)

The container will brings their full advatanges with gitlab-ci as

  * they don't take extra memory when not being runned
  * you can create a new container from a base image in a blink of eyes

Fortunately for us a Docker image already exists with all the base 
things to have a Gitlab-ci Runner in a container. [The repository](https://github.com/sameersbn/docker-gitlab-ci-runner)

If you have already docker installed you can pull the image doing 

```
docker pull sameersbn/gitlab-ci-runner:latest
```

Then you need to register your runner to gitlab-ci you can do that with

```
mkdir -p /opt/your-project-runner
docker run --name your-project-runner -it --rm \
    -v /opt/your-project-runner:/home/gitlab_ci_runner/data \
  sameersbn/gitlab-ci-runner:latest app:setup
```

it will ask you some information given to you by your instance of gitlab-ci

you can then pop a runner by doing 

```
docker run --name your-project-runner -it -d \
    -v /opt/your-project-runner:/home/gitlab_ci_runner/data \
  sameersbn/gitlab-ci-runner:latest app:setup
```

you will certainly to customize some stuff on your container
you can do so by connecting as root to it using 

```
docker exec -it your-project-runner bash
```

of course something to keep in mind is that container are meant to be stateless
which mean that outside of the directories you map with `-v` option
the other information will be deleted when you restart the container

So if you need to always create the same kind of container, you can create
your own Runner container by extending this one to fit your needs, like
it has been done in the [docker image to run test for gitlab's own source code](https://github.com/sameersbn/docker-runner-gitlab)

In a next article we will see in details how to create step by step your own
Gitlab-ci for your php/symfony project
