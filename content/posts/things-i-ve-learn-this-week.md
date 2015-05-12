+++
date = "2015-05-12T22:40:38+08:00"
draft = true
title = "Things I've learn this week"

+++


### How to send the unchanged Host header to proxied server with Nginx

Imagine you have a first nginx acting as a load balancer with this configuration

```
upstream blue  {
    ...
}

upstream green  {
    ...
}

server {
    include /etc/nginx/backend.conf;

    location / {
        proxy_pass  http://blue;
        proxy_connect_timeout   5;
    }
}


```

if you do like that, with `blue` and `green` hosting your app, your application
code will see "blue" or "green" as the http Host header, which may cause problem
when creating absolue URL

you can override this, and force nginx to send the Host given by the client to
the proxied server unchanged by adding one line like this:

```
    location / {
        proxy_pass  http://$activeBackend;
        proxy_set_header Host   $http_host; # <= this line
        proxy_connect_timeout   5;
    }


```

### How to reload PHP-fpm without restarting it

In production you may want to reload php-fpm without restarting it
in order to avoid downtime.

You can do this by sending the unix signal `SIGUSR2` to the PID of php-fpm

It may provde useful when

  * updating the php.ini
  * if you to clear the opcache
