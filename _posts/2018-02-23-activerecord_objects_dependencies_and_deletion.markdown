---
layout: post
title:      "On Dependencies and Deletion of ActiveRecord Objects"
date:       2018-02-23 19:39:26 -0500
permalink:  activerecord_objects_dependencies_and_deletion
---

One unexpected error that I encountered while working on my first Rails app was in implementing a delete feature. Basically, the ActiveRecord ORM would not allow me to delete the model instance that I wanted to delete, and threw an exception like this: 

```
Project.last.delete

SQLite3::ConstraintException: FOREIGN KEY constraint failed: DELETE FROM "projects" WHERE "projects"."id" = ?
```

This was a new error that I had not experienced before in simpler domain models. Luckily it was fairly simple to debug (thanks to Google, StackOverflow, and in the end, common sense). The issue was that the model instance that I was attempting to delete had many collections tied to it. In my case, each Project model is tied to other models:

- ```project``` has_many ```comments```
- ```project``` has_many ```rewards```

Each Comment and Reward instance that belonged to the Project instance that I wanted to delete were dependent on that Project instance. ActiveRecord was throwing an exception because if I deleted that Project instance, the model instances that rely on it, still existant, would now be referring to something that didn't exist. The Comment model instance would now belong_to... something no longer there. Logical consistency breaks down, and like a Jenga tower of interdependent blocks, if you delete one, the whole structure loses its stability.

![](https://i.imgur.com/Vh2jIMX.png)
> If interdependent ActiveRecord objects were Jenga Blocks and you were to take one out that others depended on, the structure would topple.

In order to enable deletion of my model instance, I would first need to remove the various other model instances that reference it, so that nothing would be left in the database that refers to the would-be-deleted model instance. 

Rails can handle destroying those dependent models right in the model definition:

```
class Project < ActiveRecord::Base
   has_many :rewards, dependent: :destroy
	 
	 ....	 
end
```

This would cause Rails to destroy all of the dependent records in our database in one fell swoop when the destroy action is called on the Project model instance. 

If using Foreign Keys in our database, another way to handle this is in the database migrations directly, by adding cascading deletes to the table of the dependent object(s) (the ones with the belongs_to association).

```
add_foreign_key :rewards, :projects, on_delete: :cascade
```

This effectively chains deletion of the related object instances. Once this is taken care of, ActiveRecord won't complain. 

In general, making sure that all dependent object relations are taken into account when deleting or modifying a single object instance, is a very good thing to remember when working with more complex, interrelated databases/domain models. 


