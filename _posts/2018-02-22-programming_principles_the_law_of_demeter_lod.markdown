---
layout: post
title:      "Programming Principles: The Law of Demeter (LoD)  "
date:       2018-02-22 21:15:05 +0000
permalink:  programming_principles_the_law_of_demeter_lod
---


![](https://i.imgur.com/ftD6z6u.png)
> The Law of Demeter visualized, beside The Greek Goddess Demeter, where it gets its name

The Law of Demeter (LoD) is a software development design principle proposed by Ian Holland of Northeastern University in 1987. It is named after Demeter, the Greek Goddess of agriculture, who's name means something like "Distribution-Mother," to signify a *bottom-up* philosophy of programming. Also called **The Principle of Least Knowledge**, this programming guideline states that no unit of code should have knowledge to any units outside of itself (or its immediate neightbours). 

The simple design style can be summarized by the following points:

- Each unit should have only limited knowledge about other units: only units "closely" related to the current unit.
- Each unit should only talk to its friends; don't talk to strangers.
- Only talk to your immediate friends.

![](https://www.brandonsavage.net/wp-content/uploads/2015/01/donttalktostrangers-e1421333484115.png)
> The Sign of Demeter

The notion is that each unit of code, like objects in nature, should be able to operate without assuming much of anything about any other unit of code (including its sub components). This is related to the principles of "information hiding" or "encapsulation." Having each unit of code be modular means less convuluted architecture, and thus, less modification of multiple parts of the program when one thing has to change. 

According to Wikipedia, when applied to object-oriented programs, the Law of Demeter can be more precisely called the “Law of Demeter for Functions/Methods” (LoD-F). In this case, an object ```A``` can request a service (call a method) of an object instance ```B```, but object ```A``` should not "reach through" object ```B``` to access yet another object, ```C```, to request its services. Doing so would mean that object ```A``` implicitly requires greater knowledge of object ```B```'s internal structure.

In OOP, the Law of Demeter is sometimes known as the one-dot rule:

```
user.best_friend

# is better than 

user.friends.find_by(best: true)
```

because all of the "friend"-related complexity is hidden away within the user model. This protects user (Object A)-related code from future changes to friend (Object B) functionality. For example, if the above architecture changed in such a way that "best friendship" was determined by the highest of some sort of incremented value instead of a boolean, the "two-dots" code would need to be changed everywhere its used, as opposed to the first "one-dot" implementation, which hides the complexity in *one* place (the User#best_friend method) whose definition can be changed without having to track down and update every single usage. The ```user``` model only knows about itself in relation to its neighbour, the ```friends``` model, through its own method, instead of having to procure that information  through an external method *outside* of itself (reaching *through* Object B). 

For instance, a Song model should have an instance method that introspects on itself:

```
Class Song < ActiveRecord::Base
  belongs_to :artist
	
	def artist_name
	  self.artist.name
	end
end
```

...instead of reaching through its neighbour on *every* instance of desired functionality, so you only have to make changes in one place. The object knows about itself and its relation to its neighbour, and does not extend that knowledge to any other place in the codebase (it is not nosy). 

In dealing with more complicated apps with the MVC architecture, we must also apply this principle towards *separation of concerns*. For example, protecting our views from complexity that actually belongs in our models. The View should not *directly* know about the model, just as the Model should not *directly* know about the View. The Model-related logic needs to be encapsulated in our models and only invoked when we need them, not spread out and re-written every time we need that logic.

In following this principle, we can write cleaner, more maintainable, less convuluted code, that, when needs to be changed, only needs to be changed in one or two places instead of *everywhere*. 







