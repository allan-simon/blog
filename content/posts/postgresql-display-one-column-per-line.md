+++
date = "2015-02-10T11:48:21+08:00"
draft = true
title = "postgresql display one column per line"

+++

##Problem : displaying a table with a lot of columns

You always got that table with a dozens of columns of type text and for which
the normal tabular output does not work nicely and you end up with something 
like 

```
 articleid | plop | pouet | prout | tagada |  id   | newsid  | newstype |                                                          title                                                          | something | something | something | something | something | something | something | something |      something       |      something       | something | something | something | something |          something          |    something    |                                                                                                                                                                                                                                                                                                                 something                                                                                                                                                                                                                                                                                                                 |            something            
-----------+-------+-------+-------+-------+-------+---------+----------+-----------------------------------------------------------------------------------------------------------------------------+------------+-------------+-----------+------------+-----------------+---------+--------------+---------+---------------------+---------------------+---------------+-------------+-----------+------------+---------------------------+-----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------+------------+--------------------------------+--------------+----------------------+------------------------------------------------------------------------------------+--------------+-----------+----------+----------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------+-----------+-------+------------
   1000163 |     0 |     0 |     0 |     0 |   141 | 1000147 |       10 | blablabla                                                                                           |          3 |           4 |        22 |    1000163 |               0 | 1000163 |          184 |   10028 | 2014-06-30 23:36:00 | 2014-06-30 23:36:00 |             0 |           0 |         0 |          0 |                           |           |          
```


## In MySQL

with MySQL you can replace `;` by `\G` at the end of your SQL statement, i.e turning


```
SELECT * FROM example ;
```

into

```
SELECT * FROM example \G
```


## In PostgreSQL

you need first to use `\x` and it will activate the 'one column by line' display
(use `\x` again to switch back to normal mode)

```
\x
SELECT * FROM example;
\x
```

and you will something like that 

```
-[ RECORD 1 ]------
articleid | 1000000
something     | 0
something     | 0
something     | 0
something     | 0
id        | 1
-[ RECORD 2 ]------
articleid | 1000002
something     | 0
something     | 0
something     | 0
something     | 0
something     | 2
```
