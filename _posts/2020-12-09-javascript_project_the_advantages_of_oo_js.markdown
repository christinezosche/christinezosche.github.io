---
layout: post
title:      "JavaScript Project: The Advantages of OO JS"
date:       2020-12-09 17:29:37 +0000
permalink:  javascript_project_the_advantages_of_oo_js
---


For this project, I decided to seize on my love of cooking to create an app that would make my life a little easier. I can’t count the number of times I’ve stared at the fridge, thinking, “what am I going to do with this… (whole chicken, bag of potatoes, massive jar of strawberry jam, you name it)?” 

Searching on Google or various cooking apps with ingredient names is a decent way to find recipes, if you’re willing to read and sort through dozens of options. But those searches are limited: they require that every ingredient be included in the recipe to produce a match. I thought: wouldn’t it be great if I could enter a bunch of ingredients, whatever I had in my pantry (“ground beef”, “chicken”, “lemons”, “garlic”), and the app recommended the best thing I could do with them, matching as many of those ingredients as possible. A recipe including all four of those ingredients (“Lemon Beef Sauté with Garlic Chicken”?) would be terrible. But one including as many of those ingredients as possible (but not necessarily all of them), with great ratings (“Weeknight Bolognese”) – that sounds like a winner.

To narrow down the possible recipe options (millions and millions on the internet), I turned to a reliable source: the one and only Ina Garten. One thing is certain about Ina’s cache of recipes: there are no duds.

So, with the app I’ve created, you can enter the course you’re looking to make (a starter, side, main, or dessert), the ingredients you have on hand (or want to shop for!), and the amount of time you have to cook, and it will find you the best matching Ina Garten recipe based on your requests. In Ina’s words, “how easy is that?”

![](https://i.imgur.com/sPfQ9mgh.png!)

I didn’t realize it then, but this app really allowed the advantages of object-oriented JavaScript to shine.

Creating the interactive elements for the user in JS code was fun, and rendering recipe pages through fetch calls to the API was simple enough, but the real functionality of the app lies in its ability to filter recipes based on the user’s inputs. The methods to do this could only be accomplished through object orientation.

I started by writing a Recipe class with a constructor method: 

```
class Recipe {
    static all = [];
    constructor(id, name, ingredients, time, course, ratings) {
      this.id = id;
      this.name = name;
      this.ingredients = ingredients;
      this.time = time;
      this.course = course;
      this.ratings = ratings;
      this.constructor.all.push(this)
    }
}
```

And ensuring that Recipe objects were created using data from the API when the user loads the page: 
```
document.addEventListener('DOMContentLoaded', () => {
    fetch(RECIPES_URL)
    .then((response) => response.json())
    .then((array) => {
        for (const recipe of array) {
           new Recipe(recipe.id, recipe.name, recipe.ingredients, recipe.time, recipe.course, recipe.ratings)
          }
        })
}
```

Now, with these objects created, I was able to write methods that selectively narrow down the possible recipe matches after each question is answered.

```
static filterByCourse = (search) => 
        this.all.filter(recipe => recipe.course.includes(search))
    
    static filterByCourseAndIngredients (course, ingredients) {
        let filteredByCourse = Recipe.filterByCourse(course)
        let filteredByCourseAndIngredients = filteredByCourse.filter(recipe =>
            ingredients.some(i => recipe.ingredients.toLowerCase().includes(i))) 
        for (const recipe of filteredByCourseAndIngredients) {
            recipe.ingredientsCount = 0
           for (const i of ingredients) {
               if (recipe.ingredients.includes(i)){ 
                    recipe.ingredientsCount++
                }           
            }
        }
        return filteredByCourseAndIngredients.sort((a, b) => b.ingredientsCount - a.ingredientsCount)
    }

    static filterByCourseIngredientsAndTime (course, ingredients, time) {
        let filteredByCourseAndIngredients = Recipe.filterByCourseAndIngredients (course, ingredients)
        if (time === 0) {
            return filteredByCourseAndIngredients
        }
        else {
        return filteredByCourseAndIngredients.filter(recipe => recipe.time <= time)
        }
    }

    static sortAllByRatingAndCourse (course) { 
        let filteredByCourse = Recipe.filterByCourse(course)
        return filteredByCourse.sort((a, b) => b.avgRating - a.avgRating)
    }
```
And render them on the page, the list of possible matches getting shorter each time...

![](https://i.imgur.com/DObXazdh.png)

![](https://i.imgur.com/fb0sJjdh.png)

![](https://i.imgur.com/7Kf00iNh.png)

![](https://i.imgur.com/iCr6ulfh.png)

Until only one appears:

![](https://i.imgur.com/00uRq0xh.png)

![](https://i.imgur.com/RLuqMiNh.png)

It also made it easy to render other possible matches for the user’s criteria:

![](https://i.imgur.com/Td47qrjh.png)

In sum, while JavaScript offers the ability to create convenient, interactive browser features, those features would be extremely limited if it weren’t for object orientation.

