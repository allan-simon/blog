+++
date = "2015-02-10T14:14:49+08:00"
draft = true
title = "Install docker on ubuntu 12.04"

+++

This article is a shameless summary from [http://compositecode.com](http://compositecode.com/2013/11/11/getting-docker-installed-on-ubuntu-12-04-lts/)

the steps are:

  * Updating your kernel to 3.8+ (requires a reboot)
  * Add the docker repository
  * Installing the package 

```
sudo apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring
sudo reboot
sudo sh -c "wget -qO- https://get.docker.io/gpg | apt-key add -"
sudo sh -c "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
sudo apt-get update
sudo apt-get install lxc-docker
```

And now you should be able to get docker working
