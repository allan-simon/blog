+++
date = "2015-05-24T18:26:09+08:00"
draft = true
title = "create a rest api with symfony2 Part 2: Entity,  DB Migration, CRUD"

+++

**Update 23 November 2015**: Corrected some typo, and updated bundles versions for Doctrine

In the [first part](http://allan-simon.github.io/blog/posts/create-a-rest-api-with-symfony2/) we've seen how to
create the base of a symfony2 project used to generate a REST Api.

In this part we're going to see

  * How to use the basics of JMSSerializer to serialize Entity objects
  * How to link our API with a database
  * How to create migration files to easily manage our datase over time and colleagues
  * and how to generate a full [CRUD (create/read/update/delete)](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) with form checking


#### Very important Note /!\ (can save you hours of debugging):

if at any moment you got some strange 500 errors, run

```
php app/console cache:clear
php app/console cache:clear --env prod
```

as sometimes, especially when you change the configuration files, the production environment will not update directly all the files, creating some weird errors.

### Creating the entity Article

for the moment we will limit ourselves to one entity `Article` which will contain for the moment only:


  * an id
  * a title => that can't be blank
  * a body => which is a very long text and can't be blank

(we're going to see later how to add more attributes)

So we're going to create a class in `src/AppBundle/Entity/Article.php` (the class representing
1 article, it's put to singular, so please don't name it ArticleS) That will be used by Doctrine:

```
<?php

namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 */
class Article
{
    /**
     * @var int
     *
     * @ORM\Id
     * @ORM\Column(type="integer")
     */
    protected $id;

    /**
     * @var string
     *
     * @ORM\Column(type="string")
     */
    protected $title;

    /**
     * @var string
     *
     * @ORM\Column(type="string")
     */
    protected $body;

    /**
     * We'll see later why $title and $body are put by default to ''
     */
    public function __construct($title = '', $body = '')
    {

    }

    // Note: at the opposite of the bad habits contracted by those coming
    // from the Java world we don't generate all the setters and getters
    // brainlessly, otherwise you can simply put the properties as
    // public...
    // By not creating them we're sure:
    // that nobody can set the id
    // that we understand why and how we need to add getter and or
    // setter of a property
}
```

We're going to see soon the use of the `@ORM\...`, for the moment you just need to
know it's called `an annotation` and they come from `use Doctrine\ORM\Mapping as ORM`
(hence the ORM at the beginning of the annotation)


So now we can update our controller to generate an article instead


```
      /**
       * ....
       * @return Article
       */
      public function getArticleAction($id)
      {
          return new Article("title $id", "body $id");
      }   

```

Now if you refresh the page you will an error 500

```
    "message":"Could not normalize object of type AppBundle\\Entity\\Article"
```

it's because the very basic serializer of Symfony2 does not know how
to serialize an Entity object. For that we would need to have some Normalizer
as explained here [http://symfony.com/doc/current/cookbook/serializer.html](http://symfony.com/doc/current/cookbook/serializer.html)

But as we plan to cover more complex example latters, we will go directly
with a Serializer on steroid named JMSSerializer, its documentation being [there](http://jmsyst.com/bundles/JMSSerializerBundle#usage)

to install it

```
 composer require jms/serializer-bundle
```

you may need to increase the RAM of the virtual machine temporary
to do this add the lines:

```
  config.vm.provider "virtualbox" do |v| 
    v.memory = 2048
  end 
```

and then run `vagrant reload` (it will restart the virtual machine)


once done you can activate the bundle in `app/AppKernel.php`

```
    // ...
    new JMS\SerializerBundle\JMSSerializerBundle(),
    // ...
```

And as the FOSRestBundle is already configured internally to look for the existence
of the JMSSerializer, and to use it in priority of the default symfony2 seralizer
you don't need to do anything more to get it working.

Now if you refresh the page you will see

```
{
    "title":"title 1",
    "body":"body 1"
}
```

Note: the `id` property is missing because it is currently set to null, every property
with the value null will not be seralized. In order to get an id, we need to save first
the object in database


### Generate a database migration

for the moment our database is empty (and existing as it was created during the `vagrant up`
if you check the provisionning folder , one of the file contains a task to create a database)

So in order to save our Articles in database we're going to need a table

DON'T RUN TO GOOGLE on how to create a table manually, you don't need that, instead we're going to use
a `migration` tool => Doctrine Migration because:

  * it will create automatically the SQL statement from our Entity class
  * it will create a file that our colleagues will be able to replay on their own machine (or in production) without any chance of typo or copy paste
  * it has a mechanism to be sure to not be run twice
  * it also automatically generate the SQL statment in the same file to "revert" your change if something goes wrong
  * it will always be the same command, regardless on how complicated your database changes are.
  * it knows SQL and the specific database you're using better than you.


The full documentation can be found [here](http://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html)

In order to install it, you need to edit the composer.json and add this two lines in the `require` section


```
        "doctrine/migrations": "~1.1",
        "doctrine/doctrine-migrations-bundle": "~1.1"
```

and then run `composer update`

after that activate the bundle in `app/AppKernel.php`

```
        new Doctrine\Bundle\MigrationsBundle\DoctrineMigrationsBundle(),
```

then you can create migration file using 

```
php app/console doctrine:migrations:diff
```

if everything goes well you should see

```
vagrant@vagrant-ubuntu-trusty-64:/vagrant/MyApplication$ php app/console doctrine:migrations:diff 
Generated new migration class to "/vagrant/MyApplication/app/DoctrineMigrations/Version20150524202209.php" from schema differences.
```

(the number will vary as it's generated from current date)

if you got any error message, check in priority your `app/config/parameters.yml` to see if the driver is correctly set to `pdo_pgsql` and the username password set to `vagrant` (or the ones you've chosen)

if you open the file you will it contains a SQL request, you can apply it by doing 

```
php app/console doctrine:migrations:migrate
```

Note: from now that's the only two commands you need to update your databases 99% of the time. If you need to do additional SQL request, you can always edit the file **BEFORE** committing it AND before applying it, and on the other sides, your colleagues only need to run the migrate command.

Note2: the migrate command can be run safely as many times as you want, so it can be run everytime you have a doubt that your database schema is up to date.

Note3: the migrate command will run all the migrations that are not run yet on your machine, regardless on how are missing.

### Save an article in database

For the sake of the example, for the moment we create the Article entity on the fly at each request, now we're going to persist it in database before serialization (i.e before the `return` of the controller) in order to get an Id.

Of course later we're going to move the Article creation to a dedicated API call

For that we're going to need the `Doctrine` object (i.e the PHP library used to manage database and to map database rows and object,  tables and classes, i.e the [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping)

Symfony2 already create this object and link it to the database for us, in order to access to it, we need to make our service inheriting from the the `FOSRestController` class like this 

```
<?php

namespace AppBundle\Controller;

use FOS\RestBundle\Controller\FOSRestController;

class ArticlesController extends FOSRestController
{
...
}

```


Then we can access to the `Entity Manager` (i.e the object that take care of saving and keeping in sync the PHP objects and the SQL database) by using `$this->getDoctrine()->getManager()`, and then saving our object like this


```
      public function getArticleAction($id)
      {   
          $article = new Article("title $id", "body $id");
  
          $manager = $this->getDoctrine()->getManager();
          // persist ONLY add the object to the list of object to
          // save
          $manager->persist($article);
          // only flush will actually save in database, this in order
          // to make it possible to save a lot of object in only one flush
          // (which is a LOT faster than flushing one by one
          $manager->flush();
  
          return $article;
     }

```

Then if you try to refresh the page `/api/articles/1` it will now generate you an error, saying it can't save in database because of the field `id` of Article, because we forgot to set it as autoincremented, in order to change that it's extremly simple , add the line `* @ORM\GeneratedValue` in the Entity:

```
      /*
       * ....
       * @ORM\GeneratedValue
       */
      protected $id;

```

and now you can do a `diff` and `migrate` as explained above.

Now refresh the page you should see:

```
{"id":1,"title":"title 1","body":"body 1"}
```

if you refesh again, you will see 

```
{"id":2,"title":"title 1","body":"body 1"}
```

this is because we're creating a new Article everytime.

So let's clean that


### Retrieve an article from database


#### The normal way (using the repository)

Now that we have some Article in database (accidentally created by our previous code), it's time to clean `GET /api/articles/{id}` to actually works as expected

for this you can retrieve it by doing 


```
      public function getArticleAction($id)
      {   
          $article = $this
              ->getDoctrine()
              ->getRepository('AppBundle:Article')
              ->find($id);
  
          return $article;
     }
```

The `getRepository('AppBundle:Article')` is used to retrieve the Repository, which is the object used to generate the SQL request for you, in order to retrieve data, there's one repository by table/class, in order to get the right one easily, you can simply use the string `BundleName:ClassName` so in our case `AppBundle:Article`

Then the `find` method is used to take an id, and retrieve one element, or null if it does not exists

It means that you have to handle the `404 not found` yourself (the `404` status code bein if you remember your reading of part 1, the code to say that a Resource can't be found)

for that simply add 

```

    public function getArticleAction($id)
    {   
        $article = $this
            ->getDoctrine()
            ->getRepository('AppBundle:Article')
            ->find($id);

        if (is_null($article)) {
            throw $this->createNotFoundException('No such article');
        } 
  
        return $article;
     }
```

and now checking for a non existing article id will correctly return you a 404

#### The simpler way (using the auto mapping of arguments)

As retrieving an object from the id given in parameters is a very common operation, symfony2
already have something to makes your lifes easier, if instead of giving the parameters `$id`
you directly put the parameters with the type you need, Symfony2 will automatically try to
find the article with the corresponding id (and return the 404 **BEFORE** calling your method)

so now your code will only be:

```
    public function getArticleAction(Article $article)
    {
        return $article
    }
```

much simpler uh ? and the 404 case is still handle


### Implementing GET /api/articles

In order to get all articles, the standard REST way to do that is simply to remove the `{id}` part,
which can directly be done with symfony2 by adding this method `getArticlesAction` (notice the S), and using the method `findAll()` of the repository, 

the API will then always return an array, either empty, or of Articles, serialized the same way as when you get one


```
      /**
S>     * retrieve all articles
       * TODO: add pagination
       *
       * @return Article[]
       */
      public function getArticlesAction()
      {
          $articles = $this
              ->getDoctrine()
              ->getRepository('AppBundle:Article')
              ->findAll();
  
          return $articles;
      }
```

and that's all

now calling `/api/articles` will give you


```
[
    {"id":1,"title":"title 1","body":"body 1"},
    {"id":2,"title":"title 1","body":"body 1"}
]
```

#### Important Note concerning pagination:

Pagination being a little more complex topic, we will cover it in a later part
However if you remember your reading of Part1, pagination belongs to the metadata
of the resource, so they belongs to the header of the HTTP response, which means
that when pagination will be implemented, the json returned will still looks like
that which permit

  * to implement only one parser on client side for list, regardless of paginated or not
  * permit a better flexibility and retro compatibility even if you "ooops forgot pagination" or you latter decide to remove it to simplify your API


### Implement POST /articles (creating one new article)

Note: of course never use GET to create (or edit) a resource, nor use /articles/create etc.

The goal is to be to POST a json looking like the entity

```
{
    "title": "your title",
    "body": "your body"
}
```

and if everything is ok to get a `201` (standard HTTP status code for CREATED) with the created entity

and if not to get a `422` (standard HTTP status code for `the entity is a json, but the application can't understand it as a valid article)

this time in addition to the `postArticlesAction` you need also to create a Form class (in `src/AppBundle/Form/ArticleType.php) that will do the conversion and validation for you

check [this commit](https://github.com/allan-simon/symfony2-rest-api-example/commit/61c0ca878d82151915705808a5dd20295c8c3b2c) for the list of changes but basically now we have

  * a check that the entity can't contain an empty or null body and title
  * if valid, the article is saved in database
  * if valid the article saved, which is `id` is returned
  * if non valid a list of why the article is not valid is returned

now you can test your POST in the shell by doing

```
 echo '
{
    "body" : "plop2@plop.com",
    "title" : "hello"
}
' | http POST  http://localhost:8181/app_dev.php/api/articles

```
(http is the command provided by [httpie](https://github.com/jakubroztocil/httpie) a MUST-HAVE for testing API


#### Important note

There's actually a much simpler to do, using a technique similar to the one for GET, but
unfortunately it's late at the time of writing, so I will edit this blog post
soon to include it

### Implement PUT /articles/{id}  (editing an existing article)

`PUT` is the standard way in HTTP to modify (or create) a resource for which you already
know the URL, in our case to edit, we already know the url, as it is /api/articles/{id}

So it's going to be the same for PUT, which will be a mix between GET and POST

Here it's very simple as we have already created everything in the previous steps,

we let it as an exercise to the reader (or you can directly check on github how it was implemented)

### Implement DELETE /articles/{id} (delete an existing article)

### Conclusion

Now we know how to create a Full crud which leverage as much as possible the utilities provided
to us by symfony2 and the FOSRestBundle.

We're also now able to fully manage our database over time using the Doctrine migrations tools

In the next part we will see

  * How to add functionnal testing
  * How to pre-fill the project with fake data
  * How to create relationship between Resources (Articles and Comments)
