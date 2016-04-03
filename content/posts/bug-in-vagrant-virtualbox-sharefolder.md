+++
date = "2015-08-23T03:43:56+08:00"
draft = true
title = "bug in Vagrant with Virtualbox sharefolder"

+++

## The problem: serving static files

The problem will affect you if you try to serve static files
from your vagrant folder if you use the default from virtualbox

For example you got your nginx or apache to serve css files
and when you update them, they don't get updated, and sometimes
they even come out with some garbage bytes at the end of it

<!--more-->

## The reason, a long standing bug in virtual box

after reading [this bug report](https://github.com/mitchellh/vagrant/issues/351) in vagrant repository

I've found that the underlying bug is from virtualbox, bug which has been reported [here](https://www.virtualbox.org/ticket/12597)


## The solutions: deactivate send or switch of sharefolder type


If you use Nginx or apache you can just followed the [official workaround](https://docs.vagrantup.com/v2/synced-folders/virtualbox.html)

In Nginx:

```
sendfile off;
```

In Apache:

```
EnableSendfile Off
```

If you are like me and you're serving your files from go then you have no
other choice than to swithc the sharefolder type, nfs for Unix-ish OS and
SMB for windows

A colleague of mine made this to have a transparent switch for your developers

```
def host_box_is_unixy?
  (RUBY_PLATFORM !~ /cygwin|mswin|mingw|bccwin|wince|emx/)
end


Vagrant.configure(2) do |config|

   # your stuffs

  if host_box_is_unixy?
    config.vm.synced_folder "./", "/vagrant", type: "nfs"
  else
    config.vm.synced_folder "./", "/vagrant", type: "smb", mount_options: ['ip=192.168.50.1'] #host side of :private_network
    config.vm.network "private_network", ip: "192.168.50.12"
  end
```

NFS and SMB sharefolder don't have this bug so you should be fine with that
