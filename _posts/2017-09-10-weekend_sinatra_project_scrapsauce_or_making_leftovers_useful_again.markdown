---
layout: post
title:  "Weekend Sinatra Project: ScrapSauce (or Making Leftovers Useful Again)"
date:   2017-09-09 21:26:28 -0400
---


In learning how to program, I've amassed a bunch of old code lying around from past projects and courses. A lot of these are half finished, projects with and by friends, and code that was limited by things I didn't know back then. In a word, they're leftovers. As an exercise in building a Sinatra app with data persistance, I thought instead of creating something completely new, it might be nice to revisit an old codebase and extend it, adding a database and server component with my new ActiveRecord and Sinatra knowledge. 

The basis of the app includes work by Deborah Chan, a friend and collaborator from a few years back, and bohemian coder and mentor Drew Minns. It uses the [Yummly](http://yummly.com) API to fetch recipe objects from the Yummly database, which contains all of Yummly's user-created recipes. It was abandoned some years ago and the codebase unmaintained. 

Essentially, the goal for my new version is to make use of user's leftovers as well - albeit not code, but food - by generating recipes containing their inputted food. Users can input any leftover food that they want, and the app should generate recipes based on that input. It's like placing scraps of leftover food in a blender and coming out with something tasty: thus the name "Scrap Sauce." 

The app takes input from the user via an input form: "addField."

```
<p class="manualAdd">
	<a href="#" onclick="event.preventDefault();"><i class="fa fa-plus-circle"></i> Add whatever you have lying around: </a>
	<input type="text" name="addField" id="addField" placeholder="Add ingredient">
</p>
```

It uses jQuery to target the input field, listens for keypress and pushes the ingredients to an Array (scrapSauce.ingredients) upon firing.

```
scrapSauce.ingredients = [];

scrapSauce.manualAddField = $('#addField');

scrapSauce.manualAddButton.on('click', function(){
    scrapSauce.manuallyAdd(scrapSauce.manualAddField); 
		
//This function's logic, abstracted away for the purposes of this blog post, pushes the inputted ingredient //into the scrapSauce.ingredients array

}); 
```

If input is not empty, the array elements are then fed into an AJAX call to fetch recipes containing the selected ingredients from Yummly. 

```
scrapSauce.getRecipes = function(ingredient){
    $.ajax({
        url: 'https://api.yummly.com/v1/api/recipes',
        type: 'GET',
        data: {
            format: 'jsonp',
            _app_key: scrapSauce.apikey,  //stored in a variable elsewhere
            _app_id: scrapSauce.appId,     // stored in a variable elsewhere
            allowedIngredient: ingredient
        },
        dataType: 'jsonp',
        success: function(result){
             scrapSauce.displayRecipes(result.matches); 
	    // uses JS to update the browser's DOM elements with the fetched data
        }
    });
};

if (!$.isEmptyObject(scrapSauce.ingredients)) {
    scrapSauce.getRecipes(scrapSauce.ingredients);
}
```
												
With some CSS styles applied, the front-end view and user-flow looks something like this: 

![](https://i.imgur.com/4YxiE2b.gif)

Until recently, the app was just an index.html file, some CSS styles, and a JS script file. To add persistence and extensibility I sought to refactor the codebase into the MVC framework and add a database and server-side routing with ActiveRecord and Sinatra. This meant creating a Gemfile and bundle installing Active Record, Sinatra and a bunch of other dependencies (bcrypt for password encryption, tux and capybara for tests, etc.), creating a Rake file for task automation, creating directories for my models, controllers, views, and public components, and in the end moving all of my files into the appropriate directories of this new format. 

```
├── app
│   ├── controllers
│   │   ├── application_controller.rb
│   │   ├── recipe_controller.rb
│   │   └── users_controller.rb 
│   ├── models
│   │   ├── recipe.rb
│   │   └── user.rb
│   └── views
│       ├── users
│       │   ├── login.erb
│       │   ├── signup.erb
│       │   └── show.erb
│       ├── recipes
│       │   └── show.erb
│       ├── home.erb
│       ├── layout.erb
```
> The app's new MVC architecture

The domain model was going to be pretty simple: people could sign up to the website as *Users*, who could search for, create and save many *Recipes*. A User would *have many* Recipes, and each individual Recipe would *belong to* a User.

```
class Recipe < ActiveRecord::Base
	belongs_to :user
end

class User < ActiveRecord::Base
	has_many :recipes
	has_secure_password  #bcrypt method for password encryption
end
```

After running my Rake migrations and creating my Users and Recipes tables, I needed to create the appropriate Views for registering and logging in users, as well as the routes needed to navigate the app.

All pretty straightforward. The tricky part was saving the recipes themselves. Each recipe was dynamically generated using AJAX and jQuery on the front-end, but my database and ActiveRecord instances ran on the back-end. I hadn't done it before, and it was tricky figuring out how to take information in the browser and communicate that to my database. I first tried to turn my .js script file into a .js.erb file, interpolating Ruby code with the JS. I soon figured out that wouldn't work, as the front-end elements (and corresponding variables) that I needed to save were created dynamically and instantly *after* page load, and Ruby (on the back-end) had no idea that they existed.

After many an hour stalking the virtual halls of StackOverflow, I found out that the solution was to use AJAX itself in-browser to send the front-end information to my back-end routes. 

The .save function is called when a user favourites a Recipe.

```
scrapSauce.save = function(recipe) {
        var ingredients = recipe.ingredientLines
        var dataToSave = {
            name: recipe.name,
            images: recipe.images[0],
            recipe: ingredients,
            total_time: recipe.totalTime,
            }
        $.ajax({
            url: '/recipes/new',
            dataType: 'json',
            type: 'POST',
            data: {recipe: JSON.stringify(dataToSave)},
            success: function(data) {
                alert("Successful");
            },
            failure: function() {
                alert("Unsuccessful");
            }
        });
}
```

The function first puts the desired elements of the JSON 'recipe' object that we want to save into variables, then sends the data through a post request to the route *'/recipes/new'* using AJAX. We then run browser alerts to test whether the action was successful. 

Upon success, the Sinatra route handles the sent data on the back-end and creates our Recipe object. It also updates the newly created Recipe object with a user_id, pairing it with the current logged-in User (who's id has been stored in the browser session's user_id key).  

```
class RecipeController < ApplicationController
		Recipe.create(params["recipe"]).update(user_id: session[:user_id])
end	
```

With that, recipe instantiation is achieved. The data is now persisted on the backend and exists throughout the entire app until deleted. 

![](http://i.imgur.com/xi8gDXC.gif)

The next problem was that the recipes could be viewed and deleted by any user of the app. What was needed was a way for the app to *know* when the recipe's creator was the one viewing the recipe, and allow deletion only then. We achieve this by creating *helper methods* that check current user identity. Placed in the ApplicationController (where all of the more specific object controllers inherit from), the entire app has access to these methods.

```
  helpers do 
    def logged_in?
      !!session[:user_id]
    end

    def current_user
      User.find(session[:user_id])
    end
  end
```

Now, only if the recipe object loaded in the current view has a user_id that matches the current user's user_id, will the view render the Delete button. 

```
<% if logged_in? && @recipe.user_id == session[:user_id] %>
	<form action="/recipes/delete" method="POST">
		<input type="hidden" id="hidden" name="_method" value="delete">
		<input type="hidden" name="recipe_id" value="<%=@recipe.id%>">
		<input type="submit" value="Delete">
	</form>
<% end %>
```

These methods are super helpful, used in many places in the app to dynamically change elements and permissions pages based on whether a user is logged in, authenticated, and whether the correct user is logged in.

Taking old unmaintained thought-to-be-already-finished code, I was able to use new things I've learned to add an entirely new dimension to the app. I'm pretty proud of this quick weekend hack and I think it shows off a lot of the new web technologies that I've learned so far on this learning journey. It just goes to show that whether old code, or leftover food, we shouldn't rush to throw things out - they could still make for tasty surprises. 

Watch me demo the app below: 

<iframe width="560" height="315" src="https://www.youtube.com/embed/dvmH4O8-NNk" frameborder="0" allowfullscreen></iframe>
