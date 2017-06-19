---
layout: post
title:  "Code As Metaphor: Learning Object Orientation"
date:   2017-06-19 02:07:18 -0400
---

When I started coding as a hobbyist a few years ago, I began by seeing a computer program as a list of tasks for the computer to follow. I would write the logic of my first simple programs as a series of statements to be carried out in succession, like a check-list. Information, or so-called *data*, was passed around the program ecosystem, and procedures, subroutines, or so-called functions or *methods*, were things that manipulated said data. 

In other words, the program would run top to bottom, and the aforementioned functions would manage, manipulate or iterate over the data given to them, before moving onto the subsequent lines of code in cascading top-to-bottom order. It was a mechanical way of writing code, almost like orders to the computer. Years later, I heard of a different paradigm called *Object Oriented Programming* (OOP),  though what it really meant eluded me until I began learning Ruby, a purely Object Oriented language. The implementation of Object Orientation is actually fairly simple. Conceptually, understanding the *why* of its importance, and effectively using it, is much more difficult, but when understood, is also very powerful.

![](https://10philosophycm.wikispaces.com/file/view/Platonic%20Forms%202.jpg/387085090/468x348/Platonic%20Forms%202.jpg)
> Plato's Theory of Forms

The ancient Greek philosopher Plato would speak of the 'ideal' form. There were material forms, but there were also, according to him, higher forms. Material forms exist in the physical: like various different trees scattered in a park. But in the mind, there was a so-called 'Platonic Tree' - a higher, ethereal idealization of what all trees, in general, were: the 'idea' of a tree. This metaphysical 'idea' of a tree, according to Plato, is what allowed humans to ponder the earthly material trees and categorize them as such. We can conceive of objects in our mind without having the object ever be in the room. In this way, we are really talking about the 'idea' of the object: a mental representation or model, or in philosophy-talk, it's so-called *'Platonic Form.'* Even when we speak the name of something aloud, that something may not be physically present, but the sound waves reverberating through air create the 'idea' of it. Likewise, when we write the word 'tree,' it is representative of an actual tree, not an actual tree. 

![](http://johnnyholland.org/wp-content/uploads/2011/12/aristotle-header.jpg)
> "The greatest thing by far is to be a master of metaphor." - Aristotle, looking super serious.

This meta-thinking is powerful, and is what allowed Plato to extend his reach beyond the here and now, and to conceive of the 'idea' of the atom, his theorized indivisible building blocks of the universe (from the Greek word for 'apple'), centuries before they could be mechanically proven as physically real. The 'ideas' of a tree or an atom are metaphorical representations of physical things, a way to mentally model the world around us, think about and make sense of it, and Object Oriented Programming is just that: your code as metaphor.

![](http://www.thepositiveencourager.global/wp-content/uploads/2013/05/Alan-kay.jpg)
> "I thought of objects being like biological cells, only able to communicate with messages." - Alan Kay at XPARC, on inventing Object Oriented Programming

Switching programming styles to Object Orientation is a powerful thing. Instead of seeing your programs as merely a check-list of ordered statements with methods separate from data, you begin viewing your program as a mini universe that you design to your liking. Like *our* universe, your program's universe is populated by *'Objects.'* However, they aren't physical, they're metaphors - and unlike the physical world, in this programmatic universe, if you can conceive it, you can will objects into existence. These programmatic Objects, like Plato's idealized Forms, are metaphors that model the world that you're building. Your code suddenly goes from foreign to familiar, and this is a great conceptual leap in programming, because it helps the programmer make sense of the complexity they are creating. 

Just as in the physical world, where things are comprised of both information *about* itself (for example, a tree is of a certain type, and it has leaves) and actionable behaviour (trees can turn sunlight into energy via *'photosynthesis'*, their leaves can *'grow'*), Objects in code have *data* attributes comprising information about them, as well as *methods*, which enable their behaviour. Thus, data and methods, instead of being a mesh of discrete entities, exist simultaneously *within* Objects in OOP. Methods (or messages) can then be sent from object to object, enabling this behaviour. 

In Ruby, the blueprint for a *type* of object is called a *Class*. This is like the 'Platonic Form' of trees in general, which encompass a tree's 'tree-ness,' the idea of a tree. Each individual tree, like a Birch Tree, or a Maple Tree, or a Palm Tree, are *Instances* of the Tree Class, all deriving from that one ideal, in-general idea of what a tree is. 

The following is a rather crude, silly but illustrative example of a Tree Class:

```
class Tree
   attr_accessor :type, :leaves
   def initialize(type, leaves)
       @type = type
       @leaves = leaves
   end

  def photosynthesize
      puts "I'm makin' food!"
  end

 def grow
     puts "My leaves are growing!"
 end
end
```

With it, we can now instantiate instances of 'Tree Objects' in our program, as follows:

```
evergreen = Tree.new('Maple Tree', 'Maple')
```

We now have an instance of a tree that is of type 'Maple Tree' and has 'Maple' leaves, that can grow and photosynthesize through its methods. 

Separate objects can also collaborate or interact with one another, creating more complex relationships and functionality within our programs. We can take this simple example for instance, and create a crude representation of a larger ecosystem, with a Sun Object having a *shine* method that enables it to shine onto the trees, which enable photosynthesis to occur. 

Back in Ancient Greece, Plato conceived of the idea of an atom, his then metaphor-only building block of the physical universe. Avi Flombaum, dean and co-founder of NYC's Flatiron School, often uses this as a basic but powerful example to illustrate OOP. 

An atom can be modelled in code:

```
class Atom
   attr_accessor :protons
   TABLE = {
       1 => "Hydrogen",
       2 => "Helium",
       3 => "Lithium",
       8 => "Oxygen"
   }
   def initialize (protons)
       @protons = protons
   end
   def name
      TABLE[protons]
   end
 end
```

Two instances of atoms - Hydrogen and Oxygen:

```
hydrogen = Atom.new(1)
oxygen = Atom.new(8)
```

Multiple atoms, in the right combinations, can create 'Compound Elements':

```
class Compound
  attr_accessor :atoms
  def initialize(*atoms)
     @atoms = [ ]
     atoms.map{|atom| @atoms << atom}
  end 
  COMMON_COMPOUNDS = {
     "Water" => "HHO"
  }
  def elements
    atoms.flatten.collect{|a| a.name}.sort.join.to_s
  end
  def common_name
    COMMON_COMPOUNDS.invert[elements]
  end
end
```

```
water = Compound.new(h, h, o)
```

In our code metaphor, individual atoms can collaborate to create compounds, and we now have the water needed to make our trees from earlier grow.

When creating increasing levels of complexity in software, chunking functionality and data into discrete Objects create order from chaos and allow us to find patterns and logic from what was once just 1s and 0s. 

Model your universes.
