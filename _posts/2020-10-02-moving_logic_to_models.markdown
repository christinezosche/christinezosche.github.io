---
layout: post
title:      "Moving Logic to Models"
date:       2020-10-03 01:01:18 +0000
permalink:  moving_logic_to_models
---


In our Sinatra unit, one of the main things I remember hearing over and over was “keep logic out of the controller”. It would save us work and time, they said. Fair enough, but when it came time to build my Sinatra project, that didn’t seem to be the case. Going through all my code, refactoring views and controllers, looking for repetition and places to shorten my code and then move it all to separate files… In truth, it was a lot more work than just leaving it be!

However, working on my Rails project, I quickly realized how wrong I was. Perhaps it was because this project was more complex, with more files, more relationships, more views, more controllers… refactoring to move logic to models actually saved a great deal of space and a few headaches.

For example, on the User show page, I wanted to list the user’s top five most recent trips, while also making sure that the user could only view their own page, not those of other users.

This was the first draft of what I had in the Users controller:

![](https://i.imgur.com/tVqeBJml.png)

This seemed like a prime example for some refactoring. I started by reconsidering the sorting action. That surely belonged somewhere else. What was being sorted? The user’s trips. So I headed over to the Trip model and wrote a simple sorting method:

![](https://i.imgur.com/7ypEIScl.png)

Now, I could work on applying this method to just the user’s trips. So I went to the User model and added a method using the one I just wrote:

![](https://i.imgur.com/qkWRchFl.png)

That already cleaned up a lot! But there was one more thing that could be refactored. That long stretch of code to check whether the user is accessing their own page? That’s also better in a method. After all, I might need to use it again!

![](https://i.imgur.com/D2Hl54El.png)

Finally, the user show action looked like this:

![](https://i.imgur.com/REqi9Idl.png)

Clean, neat, and easy to follow – If I happened upon this code as another developer, I would understand what it was asking without getting lost in the weeds.

Not only did this refactoring keep my code cleaner, it was actually more helpful – I found myself using these methods more than once, in views and in controllers. In fact, I was more willing to use them because they were already written and logically organized! That certainly made my app more interesting and certainly saved a few frustrating typos (and therefore hours looking for bugs) along the way.
