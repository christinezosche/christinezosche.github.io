---
layout: post
title:      "ActiveRecord Migrations"
date:       2020-07-31 18:57:10 +0000
permalink:  activerecord_migrations
---


Having just finished up the coding on my Sinatra project (or so I thought…), I took one final walkthrough of the app in my browser.

It all looked good. The links functioned as they were supposed to, the forms worked, the database updated as needed, the pages were organized and displayed properly. 

![](https://i.imgur.com/a4mOx15l.png)

But looking at this home page, something caught my eye. I noticed that my welcome message displayed the user’s username (“Welcome, Luke_Skywalker.”). Not only that, the usernames were shown in the “Pilot” column. It hadn’t stood out to me before, since my example usernames were pretty straightforward. But what if a user created a username that they didn’t want shown to the public? What if it was embarrassing? Wouldn’t it be more personal for the user – and more informative for all users, in the case of this list of flights -- to display the user’s name instead of their username? Of course it would.

But there was one problem: my database and sign up form were not set up to take in a user’s name. This was a terrible time to have this realization -- I thought I had finished coding! Would I have to start all over again, setting up my database from scratch? Perhaps, if we weren’t using ActiveRecord. Fortunately, with this gem, we can change the whole dynamic of the app with a few simple steps.

The first step is to set up a new migration to modify the existing database.

```
rake db:create_migration NAME=add_name_to_users
```

In a few seconds, we have a new migration file ready to edit. In it, we’ll define what we want to happen when this migration is run:

![](https://i.imgur.com/i4QybgWl.png)

In short, by running this migration, a column will be added to the “users” table for the “name” attribute, whose data type is a string. 

The next step is to run the migration!

```
rake db:migrate
```

![](https://i.imgur.com/aqeCKmql.png)

Success! Now, the “users” table will be storing a “name” for each user -- in addition to a “username” and “password” -- upon sign up.

![](https://i.imgur.com/GnspoyYl.png)

Great! But how do we get that information into the database? We need to modify our sign up form:

![](https://i.imgur.com/TcKk2KJl.png)

As is, there is no spot on this form to input a “name”. But the solution is as simple as adding a new line -- with an input field for the “name” attribute -- to collect this information…

![](https://i.imgur.com/iHyi2Lcl.png)

… and modifying the corresponding controller action to take that “name” input and store it when creating a new user. In this case, all we need to do is to change line 17 in our “user” controller to take in the value submitted in the “name” field of the form and assign as to the “:name” attribute of our User instance.

Before:

![](https://i.imgur.com/oy2I1SLl.png)

After:

![](https://i.imgur.com/Hvc9js7l.png)

![](https://i.imgur.com/2xIxqe3l.png)

Now, when a user signs up, their name will be stored in our database! Just storing this information isn’t enough, however. We need to change our code in a few places throughout our app so that the user’s name is displayed instead of their username.

Such as…

…lines 1 and 17 of the code for our main flights page, which welcomes the user and lists all the scheduled flights…

Before:

![](https://i.imgur.com/kjNwtSFl.png)

After:

![](https://i.imgur.com/O2oVgjQl.png)

…and line 1 of the code for the user’s home page, which shows the flights they’ve scheduled.

Before:

![](https://i.imgur.com/hLnphNMl.png)

After:

![](https://i.imgur.com/ANH43QJl.png)

And that’s all we need, right? Well, if our database were empty, that would be the case. But we have two users in our database that signed up before we made these changes. Thus, neither of them has a value for “name” recorded in the database.

```
>> han=User.find_by(username: "Han_Solo")
=> #<User id: 9, username: "Han_Solo", password_digest: "$2a$12$jnxaTYd0GovSmvEIEq4yUeKS3.y9f89NE8hLDx9a2Bf...", name: nil>
```

Here, Han Solo’s “name” has a value of “nil”. So, we have to update our users. We could roll back our database, change our data seed file, and migrate everything again -- but since there are just two users, it’s easier to just update them by hand, as follows:

```
>> han.update(name: "Han Solo")
>> han
=> #<User id: 9, username: "Han_Solo", password_digest: "$2a$12$jnxaTYd0GovSmvEIEq4yUeKS3.y9f89NE8hLDx9a2Bf...", name: "Han Solo">
```

There we have it! Han Solo’s name appears instead of nil. If we do the same for Luke_Skywalker, we’ll be all set.

Let’s check it out:

![](https://i.imgur.com/s07aUDJl.png)

![](https://i.imgur.com/tMcttYSl.png)

Thanks to ActiveRecord, we were able to change our app’s appearance -- and the data stored in our database -- quickly and easily.


