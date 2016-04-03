+++
date = "2015-05-23T23:31:15+08:00"
draft = true
title = "Create a production ready rest api with symfony2 (part 1)"

+++

What will we cover:

  * how to transform a just-started symfony project into a REST-ready project
  * how to configure and use the FOSRestBundle to help us in that task
  * How to use Doctrine migrations to easily transition your database from version N to version N+1
  * how to add functionnal tests to automatically check your API
  * how to configure and use the JMSSerializer to automagically serialize and unseralize your data into json (or xml)
  * How to plug your API to a Oauth2 service
  * mixed in between some generic advices to make your API reality proof
  * practical examples of common "tricky things" (Image uploading / pagination / association between resources etc.)

Here we suppose you're already familiar with what a REST API **exactly** means,
and I really mean the emphasis in **exactly**

for this I urge you to read these links:

   * [http://martinfowler.com/articles/richardsonMaturityModel.html](http://martinfowler.com/articles/richardsonMaturityModel.html)
   * [http://blog.steveklabnik.com/posts/2011-07-03-nobody-understands-rest-or-http](http://blog.steveklabnik.com/posts/2011-07-03-nobody-understands-rest-or-http)

   * [https://github.com/interagent/http-api-design](https://github.com/interagent/http-api-design)
   * [www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api](www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)

An API is made to resist the test of time, so you can really afford to spend one or
two hours to carefully read these links.


### Example project

All the steps described here are applied in this toy project
[https://github.com/allan-simon/symfony2-rest-api-example](https://github.com/allan-simon/symfony2-rest-api-example)

It's a Blog-like services with a REST API in json at the end it will permit


  * Multiple Authors
  * Possiblity to add/edit/delete articles
  * Possibility to comments
  * Creation of tags to category articles

<!--more-->

### Creating the project

To create the project in a repeatable and automated way,
regardless of your Operating system, we will use this base [https://github.com/we-bridge/vagrant-ansible-symfony](https://github.com/we-bridge/vagrant-ansible-symfony)
to bootstrap our project. It simply wrap the symfony2 installer
into a Vagrant machine (i.e a virtual machine), so that all the
additional things such as installing a database, a webserver etc.
are handled for you, in a dedicated environment so that you're
not polluating your own machine.

Make sure you have Vagrant installed and then do

```
git clone https://github.com/we-bridge/vagrant-ansible-symfony.git
```

edit the Vagrantfile as explained in the README, and then

```
vagrant up
```

it takes some times, and at the end you should be able to open
the following address [http://localhost:8080/app/example](http://localhost:8080/app/example) in your browser.


### Adding the FOSRestBundle

The FOSRestBundle is a bundle providing a lot of classes and method to create easily
a REST api, especially as it's part of the third-party bundles recommanded by Symfony
so in order not to reinvent the wheel we're going to use it.

The full document on how to install it and using it can be found here [http://symfony.com/doc/master/bundles/FOSRestBundle/index.html](http://symfony.com/doc/master/bundles/FOSRestBundle/index.html)
Here We're going only to talk about the step necessary to get things running

In the VM , in the symfony2 application directory run

```
composer require friendsofsymfony/rest-bundle
```

Then commit the `composer.json` and `composer.lock`


Then edit your `AppKernel.php` and add the bundle


```
...
new FOS\RestBundle\FOSRestBundle(),
...
```

and add in your `app/config/config.yml`

```
# app/config/config.yml
framework:
    # ...
    serializer:
        enabled: true
```

### Creating our first API call GET /api/articles/{id}


#### Putting all the API calls in /api


Create the file `src/AppBundle/Resources/config/api-routing.yml`
(empty for the moment)


Then in your `app/config/routing.yml`

```
mvms_api:
    type:     rest
    prefix:   /api
    resource: "@AppBundle/Resources/config/api-routing.yml"
```


#### Create the controller for all the /api/articles calls

#### Put the framework to automatically seralize as JSON the returned value of Controller

Without any change, you still need to create a `Response` object etc. you can make it even
simpler and directly the object or array to serialize by adding this to your `app/config/config.yml`

```
fos_rest:
    view:
        view_response_listener: 'force'
        formats:
            json: true
    format_listener:
        rules:
            - { path: ^/api, priorities: [ json ], fallback_format: json, prefer_extension: true }
```

We don't cover it here, but actually it's possible, without modifying your code, in one action
to generate a JSON output, a XML output, a HTML output, a RSS output depending on the extension set
or in HTTP header "Accept" header set.

so now you can add your controller in `src/AppBundle/Controller/ArticlesController.php`

with this content:

```
<?php

namespace AppBundle\Controller;

class ArticlesController
{

    /**
     * Note: here the name is important
     * get => the action is restricted to GET HTTP method
     * Article => (without s) generate /articles/SOMETHING
     * Action => standard things for symfony for a controller method which
     *           generate an output
     *
     * it generates so the route GET .../articles/{id}
     */
    public function getArticleAction($id)
    {
        return array('hello' => 'world');
    }

}
```

and then in `AppBundle/Resources/config/api-routing.yml`

```
blog_api_articles:
    type: rest
    resource: "@AppBundle/Controller/ArticlesController.php"
    name_prefix:  api_articles_
```

the fact that we've declared in the routing that this controller is of type rest
will make Symfony2 to automatically create the route

`/api/articles/{id}` restricted to the `GET` method

for more information on this consult [http://symfony.com/doc/master/bundles/FOSRestBundle/5-automatic-route-generation_single-restful-controller.html#define-resource-actions](http://symfony.com/doc/master/bundles/FOSRestBundle/5-automatic-route-generation_single-restful-controller.html#define-resource-actions)


so now if you try to access the URL  `/api/articles/1` with a `GET`  you will get a

```
{"hello":"world"}
```

Note that if you return an object, it will actually serialize it also automatically, it's not
only because we're returning an array.


### Conclusion and next part

as you can see in some simple step we now have a bundle that in two lines can implement an API
call.

In the next part we will see:

 * How to create an entity and generate a migration file to update the database schema
 * How to generate a full CRUD API to manipulate this API
 * How to create functionnal test
