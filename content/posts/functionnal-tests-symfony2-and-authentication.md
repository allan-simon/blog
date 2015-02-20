+++
date = "2015-02-20T14:28:13+08:00"
draft = true
title = "Functionnal tests in symfony2 with authentication"

+++

Let's say you have your symfony2 website, all setup correctly with its
set of functionnal tests using phpunit, and now you start to have 
pages that only authenticated users can access to. My first solution was to
create a client, make it load the login form, submit some valid credentials
and then load the actual page to test. It was working fine as the client
has a storage for cookie, it is acting like an actual browser in that case.

The problem was that the time to execute the test was much longer, which
started to become a problem has the test suite was growing.

The first trick is to follow them method adviced by symfony, which is for the
test environnement to enable the HTTP Basic Authentication method. For those
who doesn't know, the HTTP standard comme with a method to authentify a 
user-agent by directly sending the username/password in the HTTP header
(which goes along the fact that HTTP is aimed to be stateless)

in your `config_test.yml` add this (replace `NAME_OF_YOUR_FIREWALL`)

```
security:
    firewalls:
        NAME_OF_YOUR_FIREWALL:
            http_basic: ~
```


The second trick, to not have to repeat the username/password in all your
functionnal tests, is to use the [LiipFunctionnalBundle](https://github.com/liip/LiipFunctionalTestBundle)
follow the installation instruction and then make your functionnal test inherit
from their class, like this

```
<?php
//...
use Liip\FunctionalTestBundle\Test\WebTestCase;

class YourTest extends WebTestCase
{
 // your code
}
```

then you can precise in `config_test.yml` your credentials:

```
liip_functional_test:
    authentication:
        username: "XXXXX"
        password: "YYYYY"
```

finally to create an authenticated client, use this method

```
<?php
//...
use Liip\FunctionalTestBundle\Test\WebTestCase;

class YourTest extends WebTestCase
{
    public testYourFeature()
    {
        // true = "authenticated"
        $client static::makeClient(true);
        // ...
    }
}
```
