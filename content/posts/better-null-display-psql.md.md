+++
date = "2018-03-29T23:36:00+02:00"
draft = true
title = "better null display psql.md"

+++

I've been frustating recently to differenciate `NULL` and empty strings
in postgresql when requeting manually with `psql`

Turns out there's an easy way to change that.

Just use

```bash
\pset null 'Ø'
```

<!--more-->

and now 

```
 id |          email          |  first  | last
----+-------------------------+---------+-------
  1 | your-email@example.com  | Foo     | Bar
  2 | first-is-null@foo.com   |         |
```

will be

```
 id |          email          |  first  | last
----+-------------------------+---------+-------
  1 | your-email@example.com  | Foo     | Bar
  2 | first-is-null@foo.com   | Ø       |
```
