+++
date = "2015-03-28T21:05:04+08:00"
draft = true
title = "symfony create your own third party bundle part 1/4"

+++

So you've reached the point where you're reusing the same service accross
symfony application, again and again and you feel like this could even
be useful for other people. You decide that it will be great if you could
do like adults and have your own bundle.

Here is how to create one from scratch, you can find the repo here [Oauth2AccessBundle](https://travis-ci.org/we-bridge/OAuth2AccessBundle)

we cover these steps:

  1. Creating the base repo in github and composer.json
  2. Adding the base Classes
  3. Permit users of your bundle to configure it
  4. Make your bundle testable

### Create the base repo and composer.json

Well for creating the base repo, nothing different from any new repo on github.
I would however stressed out that you should chose since the beginning a license

I would recommend either

  * The MIT license, the one I personnaly chose, it permits your bundle to be
used without overthinking even by Companies with lawyer
  * The AGPL license, no the A is not a typo, like the GPL license it does
oblige people which integrate your bundle into application to make the full
application also in a GPL-compatible license and to redistribute the modified
source code with the application itself. HOWEVER, at the difference of the GPL
it cover also the case of websites/webservices/REST API (for the GPL, it only
works if the company which has made modification sells the website itself
to other customers)

You can of course chose the license your want, but be sure of what they implies.
(As said for example the GPL will not force people who use and modify your 
bundle in their own website to give you back patches etc.)

Then add a `composer.json` file with that skeleton content

```
{
    "name" : "webridge/oauth2-access-bundle",
    "type" : "symfony-bundle",
    "description" : "Symfony Oauth2 consumer bundle",
    "keywords" : ["Symfony", "Oauth2"],
    "homepage" : "https://github.com/we-bridge/OAuth2AccessBundle",
    "license" : "MIT",
    "authors" :
    [
        {
            "name" : "Allan SIMON",
            "email" : "allan.simon@supinfo.com",
            "homepage" : "http://allan-simon.github.io/blog",
            "role" : "Developer"
        }
    ],
    "minimum-stability" : "beta",
    "require": {
        "php": ">=5.3.2"
    },
    "autoload": {
        "psr-0": { "Webridge\\Oauth2AccessBundle": "" }
    },
    "target-dir" : "Webridge/Oauth2AccessBundle"
}
```

The `name` part is totally up to you but it needs to be unique accross
composer packages so most of the time `your-name-or-company/your-bundle-name`
is safe.

The `autload` part precise in which namespace your bundle classes will be
accessed

Now that we have the very base of our repository, we will see in next
post how to create a very basic service without configuration.
