+++
date = "2015-04-09T01:44:41+08:00"
draft = true
title = "doctrine2 order by count many to many"

+++

Today I found a problem quite common but tricky enough for the solution
to share it with you

### The problem

I have 2 Entities

  * User
  * Article

and a "likedByUsers" Many To Many relationship between both

Now I would like to get the Articles ordered by number of 'likes'
for an API call returning a list of Articles

if you feel smart you can try to find the solution by yourself
it's an interesting exercise about 'how much do you know doctrine and SQL'

but let me tell you, the solution is like an Egg of Columbus, once you see
it it becomes evident, but it was actually not that simple at first.

### The solution 

<!--more-->

```
// in your repository
$builder = $this->createQueryBuilder('a');
    ->select('COUNT(u) AS HIDDEN nbrLikes', 'a')
    ->leftJoin('a.likedByUsers', 'u') 
    ->orderBy('nbrLikes', 'DESC')
    ->groupBy('a')
    ->getQuery()
    ->getResult()
;
```

### Step by step

```
->leftJoin('a.likedByUsers', 'u')
```

as it's a many to many relationship, and as doctrine lazy-load by default
we need to explicitly tell him we will need the join, and the referenced
entity (users) will be refered as `u` in the other part of the query

now the interesting point here is the `leftJoin`, most PHP developers
are not familiar with the different type of Join

  * `inner join` would have taken only the articles and users who have **At least** one relationship
which mean that using this, you will not get back the articles with 0 likes
  * `left join` take ALL the row on the left side (so here articles) and the entity on the right will
be null if there's no relationship, so here that's what we want, as then the `COUNT()` will consider norelationship as `0`

```
    ->groupBy('a')
```

here it's really important to group by the FULL entity and not just `a.id`, otherwise SQL
will complain that your other fields are not inside the GROUP BY (here nothing specific as doctrine,
it's pure SQL, just that Doctrine makes your life easier by permitting you not to precise
every single field one by one


```
    ->select('COUNT(u) AS HIDDEN nbrLikes', 'a')
```

Here we have the actual interesting part 
we select a `COUNT` in order to be able to use it in the `ORDER BY` and also the
entity article itself, however if we were just doing `'COUNT(u) AS nbrLikes' , a`
then for each result, we would not directly the entity, but an array of two values
the count and then the entity (which would need additional treatment in my case
as I'm just returning an array of Article to be serialized by the REST bundle).

That's why we have `HIDDEN`, it will tell doctrine to use it for the generated query
but not to use when creating the result array, so that it returns only the entity

For those wondering if I only get the entity how to have my API call returning also
a `number_of_likes` in the JSON, I will later make an article about virtual property
of the JSM Serializer

Hope you learn something in the process,
