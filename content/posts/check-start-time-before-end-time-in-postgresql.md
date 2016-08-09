+++
date = "2016-08-10T05:48:30+08:00"
draft = true
title = "check start time before end time in postgresql"

+++

Something that a lot of people coming from MySQL does not know
about real SQL database is the possibility to have 'advanced' constraints

A very simple one here, but quite common

I have my table `Event` with a `start_time` and `end_time` and of course
I want to be sure that it can't be possible to have a `start_time`
that happens after its `end_time`. in MySQL outside of creating triggers
you will most of the time have no other way than to do this check in your
application code, and of course either it will not be put directly.
Or because of mass import of whatsoever you will always one time were this
check will not occur, and are now with a database that have inconsistent
data. And now all part of code starts to need checking that weird case.

But in PostgreSQL you can simply do this by adding a `CHECK` clause in your
create table

```sql
    CREATE TABLE event (
        id UUID NOT NULL,
        start_time TIMESTAMP(0) WITH TIME ZONE NOT NULL,
        end_time TIMESTAMP(0) WITH TIME ZONE NOT NULL,
        PRIMARY KEY(id),
        -- the important is here
        CHECK (start_time < end_time)
    );
```
Of course you can do much more than that, I invite you to check all 
the constraint you can add in reading the [documentation about them](https://www.postgresql.org/docs/current/static/ddl-constraints.html)
