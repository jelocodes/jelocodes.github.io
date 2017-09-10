---
layout: post
title:  "Weekend Sinatra Project: ScrapSauce (or Making Leftovers Useful Again)"
date:   2017-09-09 21:26:28 -0400
---


In learning how to program, I've amassed a bunch of old code lying around from past projects and courses. A lot of these projects were done with and by friends, half unfinished, with code that was limited by things I didn't know back then. In a way, they're leftovers. As an exercise in building a Sinatra app with data persistance, I thought instead of creating something completely new from scratch, it might be nice to revisit an old codebase and extend it, adding a database and server component with my new ActiveRecord and Sinatra knowledge. 

The basis of the app includes work by Deborah Chan, a friend and collaborator from a few years back, and bohemian coder and mentor Drew Minns. It uses the [Yummly](http://yummly.com) API to fetch recipe objects (in JSON (JavaScript Object Notation)) from the Yummly database, which contains all of the Yummly community's user-created recipes. It was abandoned some time ago and the codebase unmaintained. 

Essentially, the goal of my new version is to make use of a user's leftovers too: but in their case, leftover food. Once finished, a user would be able to input any leftover food that they want into the app, and would recive recipes with ingredients that match their input, so they could re-use the leftovers. Think of it like placing scraps of leftover food in a magic blender and coming out with something tasty: thus the name "Scrap Sauce." 

![](http://i.imgur.com/afWZAsV.png?2)
> The log-in screen.

The app takes input from the user via an input field: "addField." 

```
<p class="manualAdd">
	<a href="#" onclick="event.preventDefault();"><i class="fa fa-plus-circle"></i> Add whatever you have lying around: </a>
	<input type="text" name="addField" id="addField" placeholder="Add ingredient">
</p>
```

jQuery is then used to target the 'add button', which brings up the input field. It listens for a keypress and then pushes the inputted ingredients into a DOM Array (scrapSauce.ingredients).

```

// jQuery selectors: 

scrapSauce.ingredients = [];
scrapSauce.manualAddButton = $('.manualAdd a');
scrapSauce.manualAddField = $('#addField');

scrapSauce.manuallyAdd = function(inputField){

    var newContent = inputField.val();

    scrapSauce.ingredients.push(newContent);

    inputField.keypress(function(event){
        scrapSauce.updateh2(); // another function that updates the h2 with new ingredient name
        inputField.slideUp(); // animation that slides up the input field
    }
                
    // finally, fire an AJAX call to fetch Yummly recipe objects with the inputted new ingredients
    scrapSauce.getRecipes(scrapSauce.ingredients);
}); 

scrapSauce.manualAddButton.on('click', function(){
    scrapSauce.manuallyAdd(scrapSauce.manualAddField); 
}); 

```

The scrapSauce.getRecipes function feeds the scrapSauce.ingredients array elements into an AJAX (Asynchronous JavaScript and XML) call to fetch recipes from Yummly that contain those ingredients (assuming the Array isn't empty).

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
												
With CSS styles applied, the front-end view and user-flow looks something like this: 

![](https://i.imgur.com/4YxiE2b.gif)
> Search results populating.

At this point, the program is just a bunch of unorganized files. To add organization and extensibility, I refactored the codebase into the MVC (Model View Controller) framework and added a database and server-side routing with ActiveRecord and Sinatra. This meant creating a Gemfile and bundle installing ActiveRecord, Sinatra and a bunch of other dependencies (bcrypt for password encryption, tux and capybara for tests, etc.). It also meant creating a Rakefile for task automation (table migrations, etc), and creating directories for my models, controllers, views, and in the end, moving all of my files into the appropriate paths for this new, more organized format.

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
> ScrapSauce's new MVC architecture

The domain model is pretty simple: 
* People can sign up and log into the app as ***Users*** with an encrypted password (using the bcrypt Gem and the has_secure_password method for encryption and authentication)
* Users could search for, create and save many ***Recipes***. 
* A User would ***have many*** Recipes, and each individual Recipe would ***belong to*** a User.

Thus:

```
class Recipe < ActiveRecord::Base
	belongs_to :user
end
```

```
class User < ActiveRecord::Base
	has_many :recipes
	has_secure_password  
end
```

After running my Rake migrations and creating my Users and Recipes tables, I needed to create the appropriate Views for registering and logging in users, as well as the routes needed to navigate the app and send data to the appropriate views. All pretty straightforward. 

The tricky part was writing the 'save recipe' functionality: that is, enabling users to save their specific recipe search results into the back-end database for persistance. Each recipe is dynamically generated using AJAX and jQuery on the front-end, but my database and ActiveRecord instances run on the back-end. I hadn't done it before, and it was tricky figuring out how to take browser-side information and communicate that to my database. 

I first tried to turn my .js script file into a .js.erb file to interpolate Ruby code within the JavaScript itself. I soon figured out that this wouldn't work, as the front-end elements (and corresponding variables) that I wanted to save were created instantly per user search *after* the page had already been loaded, and Ruby, which is read only *at* page load, had no idea that these new elements existed at all.

After many an hour stalking the virtual halls of StackOverflow, I found my solution: use jQuery to trigger an AJAX call that sent the dynamic front-end information to my back-end routes. 

I created a scrapSauce.save function, which is called whenever a user favourites a recipe.

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

The function first puts the desired elements of the JSON recipe object that we want to save into variables, then stringifies those variables into a JSON hash and sends the data through a post request to the *'/recipes/new'* route using AJAX (For testing purposes, we also run browser alerts to test whether the action was successful or not). 

On success, the back-end Sinatra route handles the data that it receives from the front-end to create our Recipe object instantly. It also updates the newly created Recipe object with a user_id value identical to the id of the currently logged-in User by using the browser session's user_id value (a feature that was activated in the ApplicationController by editing the Sinatra::Base config settings).

```
class RecipeController < ApplicationController
		Recipe.create(params["recipe"]).update(user_id: session[:user_id])
end	
```

With that, recipe instantiation is achieved. The data is now persisted on the back-end and exists throughout the entire app until deleted. 

![](http://i.imgur.com/xi8gDXC.gif)
> Recipe instantiation in real time.

The next problem was that at this point, all recipes could be deleted by *any* user of the app. What was needed was a way for the app to *know* when the user viewing the recipe was also it's creator, and allow deletion only then. This was achieved by creating 'helper methods' to check current user identity. Because all other controllers inherit from the ApplicationController, the methods should be written in that controller so all other controllers have access to them.

```
helpers do 
  def logged_in?
    !!session[:user_id]  #if a session id exists (truthy), it means a user is logged in
  end

  def current_user
    User.find(session[:user_id]) #we find the User in the database that has a matching user_id  
  end
end
```

Now with some .erb logic in the views, the view will only render the Delete button if the recipe object currently loaded has a user_id value that matches the current logged in user's user_id value (once again, stored in the browser's session). 

```
<% if logged_in? && @recipe.user_id == session[:user_id] %>
	<form action="/recipes/delete" method="POST">
		<input type="hidden" id="hidden" name="_method" value="delete">
		<input type="hidden" name="recipe_id" value="<%=@recipe.id%>">
		<input type="submit" value="Delete">
	</form>
<% end %>
```

These methods are super helpful, used in many places in the app to dynamically change elements and permissions on pages based on whether a particular user is logged in and authenticated.

Taking old code, I was able to use new things that I've learned to add an entirely new dimension to the app, and make it more robust and complete. I'm pretty proud of this quick weekend hack and I think it shows off a lot of the new web technologies that I've learned so far. It just goes to show that whether it's old code, or leftover food, we shouldn't rush to throw things out - they could still make for tasty surprises. 

Watch me demo the app below: 

<iframe width="560" height="315" src="https://www.youtube.com/embed/dvmH4O8-NNk" frameborder="0" allowfullscreen></iframe>
