---
layout: post
title: "Back to basics: psql"
date: 2019-03-14
categories: blog
tags: psql, postgres, sql
excerpt: >
  The right tool for the right job, right? Well let me introduce you to psql.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/psql-basics">thoughtbot.com/blog/psql-basics</a>.
</div>

Have you heard the phrase that goes something like this?

> if all you have is a hammer, everything looks like a nail

Well, it turns out it is known as the [law of the instrument]. And when I first
started programming, I learned Ruby on Rails. So Rails was my hammer.

Over time, I started acquiring more tools for my tool belt. But for some reason,
when it came to interacting with a database, I kept using the hammer. And it's a
good hammer!

Using `rails console` and `ActiveRecord` is easy and very convenient.
Unfortunately, it has limitations when dealing with more complex queries or when
needing to get metadata, such as a table schema. There are many GUI tools out
there, and I hear many of them are quite good, but I prefer staying on the
terminal when I can. What is one to do?

Get another tool for the belt!

A few years ago I started using `psql`, an interactive terminal for Postgres,
and let me tell you, it's the right tool for the job!

Let me share a few easy ways to get started with it.

[law of the instrument]: https://en.wikipedia.org/wiki/Law_of_the_instrument

## Connect to a database

<kbd>psql -d app_dev</kbd>

If you don’t know the exact name of your database (surprising how often that
happens to me), you can list all of the available ones:

    # enter interactive terminal
    $ psql

Once in `psql`, run `\l` to list all available databases and `\c database_name`
to connect to one:

    # list databases
    [local] user@user=# \l

                                                List of databases
        Name      |     Owner     | Encoding |   Collate   |    Ctype    |   Access privileges
    --------------+---------------+----------+-------------+-------------+-----------------------
     app_dev      | postgres      | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     app_test     | postgres      | UTF8     | en_US.UTF-8 | en_US.UTF-8 |


    # connect to the one you want
    [local] user@user=# \c app_dev

    You are now connected to database "app_dev" as user "user".

## List tables in the database

Once you've connected to the database you need, you can get a list of all the
tables with `\dt`:

    # list tables
    [local] user@app_dev=# \dt

                           List of relations
     Schema |                Name                | Type  |  Owner
    --------+------------------------------------+-------+----------
     public | incomes                            | table | postgres
     public | addresses                          | table | postgres
     public | comments                           | table | postgres
     public | posts                              | table | postgres
     public | customers                          | table | postgres
     public | documents                          | table | postgres
     public | payments                           | table | postgres
     public | users                              | table | postgres

## Execute a query

Remember SQL? Yeah, you can do that:

    [local] user@app_dev=# SELECT * FROM users LIMIT 1;


    -[ RECORD 1 ]-----+-------------------------------------
    id                | 1
    email             | user@example.com
    name              | Test User
    store_id          | 23
    inserted_at       | 2019-01-31 20:29:32
    updated_at        | 2019-01-31 20:29:32

    Time: 7.153 ms

## Inspect a table's schema

Before I used `psql`, inspecting a table's schema usually required me to go to
the `schema.rb` file in a Rails application. But I think `psql` has something
better: with a single command, we can not only inspect a table's schema, but we
can also see all the indexes defined on that table, its foreign key constraints,
and even which other foreign keys reference our table! Test it out with `\d
table_name`:

    # describe table (also works with view, sequence, or index)
    [local] user@app_dev=# \d users

                                                Table "public.users"
          Column       |            Type             | Collation | Nullable |              Default
    -------------------+-----------------------------+-----------+----------+-----------------------------------
     id                | bigint                      |           | not null | nextval('users_id_seq'::regclass)
     email             | character varying(255)      |           | not null |
     name              | character varying(255)      |           | not null |
     store_id          | bigint                      |           | not null |
     inserted_at       | timestamp without time zone |           | not null |
     updated_at        | timestamp without time zone |           | not null |
    Indexes:
        "users_pkey" PRIMARY KEY, btree (id)
        "users_email_index" UNIQUE, btree (email)
    Foreign-key constraints:
        "users_store_id_fkey" FOREIGN KEY (store_id) REFERENCES stores(id)
    Referenced by:
        TABLE "comments" CONSTRAINT "comments_user_id_fkey" FOREIGN KEY (user_id) REFERENCES users(id)

## Find help and quit

Of course, one of the most important things to know when using command-line
tools is how to find help. This one is easy, just ask:

    [local] user@app_dev=# \?

That will give you a tremendous amount of information for all possible actions
you need to take.

And of course, it's nice to know how to quit out of the interactive shell! No
need to go to [stack overflow], even though I hear it's [quite popular] for
learning how to quit things.

    [local] user@app_dev=# \q

If you’re interested in learning more, the [postgres guide] has a good tutorial.
And if you really get into it, check out how you can [customize the psql
prompt]. Enjoy!

[quite popular]: https://stackoverflow.blog/2017/05/23/stack-overflow-helping-one-million-developers-exit-vim/
[stack overflow]: https://stackoverflow.com/questions/9463318/how-to-exit-from-postgresql-command-line-utility-psql
[postgres guide]: http://postgresguide.com/utilities/psql.html
[customize the psql prompt]: https://thoughtbot.com/blog/improving-the-command-line-postgres-experience
