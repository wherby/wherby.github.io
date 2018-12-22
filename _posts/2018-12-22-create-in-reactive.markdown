---
layout: post
comments: true
title:  "Create in reactive service"
date:   2018-12-22 13:22:16 +0800
categories: jekyll update
img: crud.jpg # Add image post (optional)
tags: [create, reactive service, crud]
---


```
“Factories incidentally if you're used to the term
CRUD -- Create, Read, Update, Delete -- factories or
the C in CRUD; they are the create. Repositories are similar to factories
but instead of abstracting away creation they abstract away the retrieving of
existing objects. Factories are used to get new objects. Repositories are used
to get or modify existing objects. Again if we use the CRUD terminology -- Create,
Read, Update and Delete -- then repositories are the RUD. They are the Read, the
Update and the Delete.” 


```
 ----From Domain abstractions in Reactive Architecture: [Domain Driven Design](https://courses.cognitiveclass.ai/courses/course-v1:Lightbend+LB0103ENv1+2018/courseware/514486c945d8469392eaba7fa84418e9/f14dbaff60be4607882ad83f68fa6d7a/)

So why the create operation is missing in repository when you want to create a reactive software. 

Let’s begin from a case using mysql database with Slick:
```sql
create table `users`(
   `id` BIGINT NOT NULL AUTO_INCREMENT,
   `email` VARCHAR(128) NOT NULL,
   `role` VARCHAR(128) NOT NULL,
   PRIMARY KEY (`id`)
)
```

And the DAO code will be like:
```scala 
  private val queryById = Compiled(
    (id: Rep[Long]) => Users.filter(_.id === id))

  override def lookup(id: Long): Future[Option[User]] = {
    val f: Future[Option[UsersRow]] = db.run(queryById(id).result.headOption)
    f.map { maybeRow => maybeRow.map(usersRowToUser) }
  }

...

  override def create(user: User): Future[Int] = {
    db.run(
      Users += userToUsersRow(user)
    )
  }
```

Then the create operation will break the reactive behavior: the operation need the synchronous check:

1.  If use auto increase value for primary key as the example above, when creating operation event are triggered several times, because the creating 
operation is not idempotent operation, then there will create several accounts for same user. 

2.  But what’s if change the primary key to the uuid key instead of auto-increased key, as below:

``` sql
create table `users`(
    `id` VARCHAR(36) NOT NULL,
   `email` VARCHAR(128) NOT NULL,
   `role` VARCHAR(128) NOT NULL,
   PRIMARY KEY (`id`)
)
```

If create operation is triggered by same event for several times, only one creation operation will be success. While there may introduce another issue:
```scala
override def create(user: User): Future[Int] = {
    db.run(
      Users += userToUsersRow(user)
    )
    }
```

When we trying to create same user, then the creating operation will not return Future[Int] as it claimed, actually the exception will be thrown. Because of create same primary key in database.


How to mitigate the create operation in reactive system, both keeping the create operation and reactive feature. The answer is simple: let the create operation has idempotent behavior.




[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
