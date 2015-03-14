+++
date = "2015-03-14T18:21:01+08:00"
draft = true
title = "migrate down with doctrine migrations"

+++

if you're using Doctrine in your Symfony2 project, you're certainly using the
excellent [Doctrine Migration Bundle](http://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html)
but you may have seen that documentation is not staging clearly how to migrate down

for that:

```
php app/console doctrine:migrations:status

```

take the number in "Current Version" (format YYYYMMDDHHMMSS)

and then run 

```
php app/console doctrine:migrations:execute YYYYMMDDHHMMSS --down
```

that's all folks !
