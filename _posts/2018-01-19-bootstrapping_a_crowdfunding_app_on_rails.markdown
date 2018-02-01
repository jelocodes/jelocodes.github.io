---
layout: post
title:      "Bootstrapping a CrowdFunding App on Rails"
date:       2018-01-19 11:56:25 -0500
permalink:  bootstrapping_a_crowdfunding_app_on_rails
---


Crowdfunding is a concept that I've loved from the moment that I've heard of it. It's a bunch of small people coming together to create something bigger than themselves. It's great. Except for the not so great parts. The not so great parts being when a project is funded, but the product never reaches the backers, or when it does reach the backers but not as advertised (see: broken and unfinished). This is in part due to the fact that, aside from basic security checks that the platform initially places on its users, a big part of the crowdfunding model relies on trusting the person that you're giving your money to to deliver, and unfortunately, with the lax accountability of these platforms, some people abuse that trust. I got to thinking about potentially better ways of doing this.

There's a relatively new technology called Ethereum that leverages Blockchain technology (see my Ruby gem [cryptocompare](http://fullstackfollies.com/2017/07/17/crypto_compare_ruby_gem/)) to create programmatic contracts that execute based on whether certain conditions have been met. I thought it could be a pretty interesting exercise to apply smart contracts to crowdfunding. For example, unless certain conditions are met (say, certain deliverables are produced), the funding won't be released to creators. This way, backers (and their funds) are safe until the product is delivered. It's still fuzzy, but the basic idea is there, but before we code the actual smart contract functionality, we need to build the basic web app. I thought it'd be a pretty fun exercise to try to bootstrap the basic functionality of the crowdfunding app.

For this blog post, I'll be focusing on the first part of creating the app, the model and database relations of the various components that are needed for its basic CRUD functioning, focusing on the trickier (more interesting) parts.

![](https://i.imgur.com/7Dr6EVd.png)
> Meet ThingFunder (it's a working title...)

I opted to use Rails for the backend, as coding with Ruby makes me happy (Minswan). The first thing I needed to do was decide what models (and their relations) were needed in the domain. At its most basic, a crowdfunding app would need: 

***Users:*** who could both create their own projects and fund the projects of others.

***Projects:*** self explanatory.

***Rewards:*** each project would have rewards that users could avail depending on size of their pledge.

***Comments:*** so users can talk about projects, and voice complaints or compliments. Each comment would belong to a user (who left the comment), as well as to a project (that the comment pertains to).

***Categories:*** Each project would fall under specific categories.

The Model Relationships are as follows:

**User:**
* has_many :projects
* has_many :comments
* has_many :user_rewards
* has_many :rewards, through :user_rewards

**Comment:**
* belongs_to :user
* belongs_to :commentable, polymorphic: true
* has_many :comments, as :commentable

**Reward:**
* belongs_to :project
* has_many :user_rewards 
* has_many :users, through :user_rewards

**Project:**
* belongs_to :user
* has_many :rewards
* has_many :comments, :as: :commentable
* has_many :project_categories
* has_many :categories, through: :project_categories

**Category:**
* has_many :project_categories 
* has_many :projects, through: :post_categories

**Join-tables:**

* user_reward
* project_category


Once those were conceptually in place, I moved on with creating each model table and migrating them into the database. 

**Comments and Polymorphism:**

While most of the tables' columns and relations were pretty easy to conceptualize, the commenting system was tricky. I imagined users to be able to comment on projects, but also, to be able to comment on other comments, so that users could actively reference each other and start mini threads. Although I was told there were lightweight Gems that could do this better, and that a hand-coded solution may not scale well, I decided to try to build the functionality from scratch for learning's sake. 

Instead of creating multiple models for comment type which would have been a lot more work and data bloat, I used something called a polymorphic association for just one model type: the Comments model. 

![](https://i.imgur.com/RL1ol11.png)

As you can see, a comment belongs to a user, and itself can have many comments. It also belongs to ```commentable```, which is a polymorphic association... commentable can be multiple things (a post, or another comment). Onto the migration:

![](https://i.imgur.com/iEtJ0cp.png)

Since a column belongs to a User, there is a column set for the user's foreign key. That's pretty standard. However, what's interesting is a column called ```"commentable_id"``` which is set to store the ID of the object that the user is commenting on. This creates a relation between the comment and the commentable thing that was commented on. To further specify what that commentable thing is, we also have a column called ```"commentable_type"```, indicating the type of object we're commenting on (whether it's a Post or another Comment).

The result is that projects and comments are both commentable:

![](https://i.imgur.com/Irpbm3G.png?1)

Pretty cool.

**Project Creation and Form Nesting:**

Creating the projects themselves is fairly straightforward. They need certain key things, including descriptors like an 'About' and 'FAQ' sections, as well as 'Categories' and 'Rewards.' The latter columns are a little bit more complex because categories and rewards are objects in and of themselves. Therefore, there needs to be a way to create those separate objects during the same process of creating a Project.

For Categories, users are allowed to either pick an already existing category, or create a new category of their choosing. As you can see, the form handles building categories directly onto the project itself, chaining their creation onto its parent. 

![](https://i.imgur.com/dnkdyl3.png?1)

which results in...

![](https://i.imgur.com/i0c40Ey.png?1)

The form is submitted, and the project is created, but not published (there is a column in the model called published that takes a boolean, not assigning itself to true until rewards are added).

The next screen is the second part of the form, and rewards are created through a nested form within the form. Rewards are created with their appropriate information (name, pledge requirement and description) and linked to the current yet-to-be-published project by supplying the form with a hidden field and value corresponding to the current project's id.

![](https://i.imgur.com/5kbcoPT.png?1)

The result, once all the fields are filled up and the project is created, is a project object with its corresponding categories objects, as well as its corresponding rewards objects, created and linked to itself via nifty ActiveRecord relations.

**Dependencies and Deletion**

After all of that, there was one more thing I knew I wanted to add to complete the basic app's functionality: a way to delete projects. I decided to pop into a rails sandbox with the handy ```rails c``` command in my console to delete a project, just to test out the command. What followed was... unexpected.

![](https://i.imgur.com/wU1JmOV.png?1)

The delete transaction was rolled back with an error citing ```InvalidForeignKey```... this was something I had not dealt with before, and it took awhile debugging to find out the issue. From a bevy of StackOverflow and Google saviours, I found out that one simply can't delete an ActiveRecord instance if that instance has others depending on it. That is to say, if other model tables have a foreign key pointing to the object you are trying to delete. In my case, many things are dependent on the project model: the rewards model tables and comments model tables for instance, both point to project tables using a foreign key column. Basically, ActiveRecord doesn't allow me to delete something that other things depend on for its associations to make sense.

Though there are many ways to solve the issue, the easiest one in my situation was to specify the ```dependent: :destroy``` option on the Project model's associations. This would tell ActiveRecord to allow the deletion of the desired project, and fixing any association mishaps by *also* deleting any of the objects dependent on it. So, now we can delete a project, and at the same time, delete all of the reward and comment objects associated with said project. 

![](https://i.imgur.com/mtEdVyg.png?1)

With some [Devise](https://rubygems.org/gems/devise/versions/4.2.0) authentications and validations for the forms, the basic skeleton of the app is working and all set up. See below for a quick demo (and some troubleshooting) of its functionality.

<iframe width="560" height="315" src="https://www.youtube.com/embed/drreRyb84PE?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Pretty satisfying bootstrap of a crowdfunding platform using Rails! We'll see if I eventually return to the project to add the actual blockchain crowdfunding component... Maybe I can crowdfund it?

