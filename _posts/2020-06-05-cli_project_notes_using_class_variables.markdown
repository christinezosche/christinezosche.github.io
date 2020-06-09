---
layout: post
title:      "CLI project notes: Using instance variables"
date:       2020-06-05 15:00:22 -0400
permalink:  cli_project_notes_using_class_variables
---


In this CLI data gem project, we were asked to write a Ruby gem. In the context of programming, I always thought the word “write” was somewhat of a stand-in word, basically meaning “create using text”. What I came to realize in this project, however, was how apt the word “write” is to describe the process of creating a program. Like writing an essay, writing an application involves outlining, a first draft, and revisions. 

In this blog, I’ll be focusing on a particular revision (or, in technical language, refactoring) that I encountered after writing the “first draft” of my Welcome to Hogwarts CLI gem.

The gem centers on four instances of a House class (Gryffindor, Hufflepuff, Ravenclaw, and Slytherin, of course) created using data pulled from an API. After the CLI is initialized, the user is asked if they would like to proceed to Hogwarts’ Great Hall for the Sorting Ceremony (to be sorted into their new house). If the user responds, “Let’s go!”, the “sort” method is called. In my first draft of the gem, the method was written like this:

![](https://i.imgur.com/JWEbf0sl.png)

The user’s house is stored in the “sorted_house” variable.

After being sorted, the user is asked if they would like more information on their house. If their answer is “Yes”, a method to print this information is called:

![](https://i.imgur.com/IZf0c5ll.png)

When conceptualizing this gem, I wanted to add some personalization to a farewell message when the user chooses to exit the program – in this case, I wanted to write, “Have a great term, Gryffindor!” if the user was sorted into Gryffindor, or “Have a great term, Slytherin!” if sorted into Slytherin, etc. Therefore, the method used to print the farewell message needed to remember the house the user was sorted into. In my first pass at writing this exit method, this was achieved by passing in the sorted_house variable created in the “sort” method as an argument, as follows:

![](https://i.imgur.com/MtjlUgFl.png)

However, this approach created one (rather large) problem. As the CLI became more complex, the sorted_house variable needed to be passed into almost every method for the final farewell message to print as intended:

![](https://i.imgur.com/1Nq6Psyl.png)

This was clunky and confusing. The sorted_house variable wasn’t even being used in most of these methods – it was just being passed around so it could be used in the exit method. This isn’t DRY. More importantly, if the application became even more complex, or if we needed to pass in other variables into these methods, this code would get impossibly confusing very quickly. There must be a better way! There is, and it’s one of the great things about Ruby object-oriented programming: an instance variable.

The previous variable, sorted_house, had a limited scope – it could only be used within the method in which it was created, which is why it had to be manually passed into every other method in the CLI class. An instance variable, however, has a broader scope -- it is accessible to all methods in our CLI object, without needing to pass it into every method. Perfect!

Therefore, I assigned a new variable in the “sort” method: @sorted_house.

![](https://i.imgur.com/31VoSMZl.png)

Then, I rewrote the other methods so that there was no need for them to take in the “sorted_house” variable as an argument.

![](https://i.imgur.com/5Oixowsl.png)

Finally, I rewrote the "normal_exit" method to utilize the @sorted_house variable.

![](https://i.imgur.com/fN0Rmn8m.png)

Voila! Now, the user’s sorted house is stored in the @sorted_house CLI instance variable, accessible to all methods within the CLI object. If we instantiate a new CLI instance and run the “sort” method, @sorted_house will change to whatever house this new user is sorted into.

![](https://i.imgur.com/jlUHyQCl.png)







