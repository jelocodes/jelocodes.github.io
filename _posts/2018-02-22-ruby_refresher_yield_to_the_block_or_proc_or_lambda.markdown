---
layout: post
title:      "Ruby Refresher: Yield To The Block (...Or Proc, or Lambda)"
date:       2018-02-22 01:01:14 -0500
permalink:  ruby_refresher_yield_to_the_block_or_proc_or_lambda
---


![](http://i.imgur.com/mLY2X1y.png!)
> Lambda vs. Proc at a glance


Ruby was built with the goal of having the programmer enjoy themselves while programming, but there are some features in the language that, if not properly internalized, can go unappreciated, under-used, or mis-used by many. In first learning Ruby, one of the most interesting but most glossed over features for me, was the block, and its peers: the proc, and lambda.

Of course, we know what a block is. Just a chunk of code between braces, ``{ }``, or between ``do/end`` keywords that can be passed to a method, similar to how arguments are passed into methods. You can make your own methods, or use some of Ruby's out-of-the-box methods, like iterator methods, that can be called with a block, to ```yield``` data to that block to operate on.

Example:

```
def demonstrate_block(number)
    yield(number)
end

puts demonstrate_block(1) do |number| 
    number + 1
end 

#=> 2
```

In that example, we created a method called #demonstrate_block which accepted one argument, ```number```, the method body yielding that argument to a block. The *yield* key word enables the method to accept a block, essentially passing in whatever code is in your method to the provided block. It stops the code from executing inside your method, and instead passes the information to your provided block, which then executes. In this case, we pass in our number into the block, which adds 1 to it. This is pretty powerful when you consider that you can put anything in a block and yield whatever other logic or data into it. This is exactly what iterator methods do, like #each. You don't really stop to consider that there is additional logic inside the method itself that is fetching the collection data for your block to do whatever it wants with. In that way, blocks are very good for encapsulation. 

A block is basically all what a proc or lambda is, but they're two slightly different variations of a block.

**A Proc, Approximately**

Whereas a block is just a block of code that you can pass in with different methods, a proc, short for procedure, is an object. A poc object is a block of code that has been *bound* to a set of local variables. Once bound, the code may be called in different contexts and still access those variables. 

Example:

```
def gen_times(factor)
    return Proc.new { |n| n * factor }
end
```

What we're doing is defining a method called #gen_times that takes in an argument of 'factor.' The method body returns a Proc object, that takes in an argument 'n' and multiplies it with the the argument 'factor.' Notice that #gen_times does not run the code inside itself, but instead simply *returns* it. What this effectively does is, whenever #gen_times is invoked with a 'factor' argument, that argument is bound to the proc inside of it. If you invoke #gen_times with an argument of '5', that method instance will now forever have a proc inside of it with the block of code ```n * 5```, the 'factor' variable forever bound to 5. 

You can invoke the code inside of a proc with .call.

```
times5 = gen_times(5)

times5.call(10) 

#=> 50

times5.call(5)

#=> 25
```

As you can see, if you place this particular instance of #gen_times in a variable and call that variable with an argument, the proc inside of it uses the passed in argument to execute with the previously bound '5'. We've essentially created a method template that encapsulates specific logic with the power to spawn multiple versions of itself. This is called a ***closure***, and they're very powerful tools.

**The (half-)Life of a Lambda**

A lambda is the eleventh letter of the Greek alphabet (λ), and is used a constant representing radiactive decay in a half-life equation (Gordon Freeman fans may also be familiar with it). In programming, a lambda is also used to refer to anonymous methods, that is, methods without a name. A lambda in Ruby is an anonymous method (that is also an object) that you can pass paremeters into via pipes:

```
lambda do |string|
    if string == "try"
		   return "There's no such thing"
		else 
		   return "Do or do not."
		end
end 
```

Looks familiar? That's because a lambda, when broken down, is just a proc. In fact, it is literally just another instance of the Proc class. We can put a lambda, that is, a nameless method that encapsulates a block of code, in a variable like so:

```
increment = lambda { |num| puts num + 1}
```

And now that functionality is available to us in that variable.

```
increment.call(2)
#=> 3
```

**So, what's the difference between a block, lambda and proc? **

***1. A block is just a chunk of code, a proc is an object:***

A block is just a piece of code that's part of a method syntax. It doesn't mean anything unto itself, and you can't call different methods on it (such as .call). A proc is a full Ruby object, meaning you can create instances of it, call methods on it, return itself, etc.

***2. You can only include one block per method, but you can include multiple procs:***

Because a block is not an object in and of itself and can only be passed in via a specific syntax in method definitions, you can only ever pass in one block per method. Because procs are objects,many of them can be encapsulated in variables and passed in as many as you'd like depending on your needs:

```
def multiple_procs(proc1, proc2)
  proc1.call
  proc2.call
end

a = Proc.new { puts "One proc" }
b = Proc.new { puts "Two procs" }

multiple_procs(a,b)
```

***3. Although Procs and Lambdas are both Proc Objects, they differ in:***

*a) argument specificity:* Lambdas care about how many arguments are passed into it, while Procs don't 
*b) returning:* The return keyword in a Lambda triggers the code just outside of where its declared, while the return keyword in a Proc triggers the code outside of the method where the proc is executed (see image up top)

The specific type you use depends on your use-case, but now you hopefully know the differences, and perhaps more illuminating, the similarities, between these three very cool Ruby features.
