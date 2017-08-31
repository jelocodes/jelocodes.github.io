---
layout: post
title:  "Trust Data, Not Lore: SQL Joins Visualized"
date:   2017-08-30 19:59:13 -0400
---

![](https://68.media.tumblr.com/52c04c960042f7e614e36fce0833f995/tumblr_oow3quxpdg1swxsyto2_500.jpg)
> Star Trek The Next Generation giving sound life advice.

Quoting a very cool [YouTube video](https://www.youtube.com/watch?v=yWO-cvGETRQ) by Kurazgesagt, *"When it comes down to it, information is nothing tangible. It's typically understood as the property of the arrangement of particles."* 

What this means is, a bunch of carbon atoms organized one way can create coal. The very same carbon atoms arranged another way can create diamonds. Zoomed in, the atoms are the same. What changes is the information. Our mental representation of physical things (e.g. a diamond or coal) is also information, and the process by which these *thoughts* manifest in the first place is also, in the end, different neuronal activity in the brain, which, once again, are just the arrangement of particles, by our definition, also information. 

In a way, the whole universe is just a collection of information, and how we perceive ourselves in it, is a product of our ability to see the *context*, or interrelatedness, or grouping, of said information. In this frame of reference, zoomed out, all of the ideas, philosophies, friendships, technology, and things that exist in this blip of space-time on planet Earth, and even Earth itself, can be seen as just different information, interrelated, arranged, and interpreted, in different ways (The carbon particles in any configuration are objectively just that, carbon particles, but to be conscious is to know that we, as humans, tend to burn one configuration and wear the other in a ring). 

From the neurons in our brains that imagine these objects and interrelations, to the clusters of stars and galaxies that form our universe, to well, the internet, all complex systems seem to converge to form similar patterns when viewed macroscopically.

![](https://lh3.googleusercontent.com/-Oscj5uMcXjI/VZJJviwnVhI/AAAAAAAAC6U/5tgUX2CHFUc/w530-h502-p-rw/FB_IMG_1435602576152.jpg)

One philosophical view could be that it's all just information out there, and SQL (Structured Query Language) gives us a way to programmatically organize and see connections between different tables or groupings of information. That's pretty cool, especially when we zoom back in and realize that our job as developers is to be masters of information and data. We create *context* and map *relationships* out of data (information) in order to bring function and order to our software applications. 

Many novice programmers, including myself starting out, get confused when first learning SQL and how exactly to query specific information from tables in a database. Tables represent sets of information about specific domains, and the results of 'join queries' in SQL represent the interrelatedness of elements between these domains.

From Ligaya Turmelle's great [blog post](http://www.khankennels.com/blog/index.php/archives/2007/04/20/getting-joins/), we have our example universe of information: two data sets visualized in the form of circles in a venn diagram, each circle representing a table (Table 1 (T1) and Table 2 (T2)) and all the records within:

![](http://www.khankennels.com/blog/wp-content/uploads/2007/04/basicvenn.png)

```
id name       id  name
-- ----       --  ----
1  Coal     1   Picard
2  Spot     2   Coal
3  Diamond  3   Neuron
4  Jordi    4   Diamond
```

The following are some typical 'join queries' one can use in SQL to query different matching records from each table, visually explained.

An **inner join** query only returns records with matches in both tables. 

![](http://www.khankennels.com/blog/wp-content/uploads/2007/04/venn1.png)

```
SELECT * FROM TableA
INNER JOIN TableB
ON TableA.name = TableB.name
id  name       id   name



1   Coal     2    Coal

3   Diamond  4    Diamond

```

An **outer join** query returns all the records from T1 and T2, with matching records from both sides where available.

![](http://www.khankennels.com/blog/wp-content/uploads/2007/04/outervenn.png)

```
SELECT * FROM TableA
FULL OUTER JOIN TableB
ON TableA.name = TableB.name
id    name       id    name



1     Coal     2     Coal

2     Spot     null  null

3     Diamond  4     Diamond

4     Jordi    null  null

null  null     1     Picard

null  null     3     Neuron
```

A **left join** query returns all of the records in the first (left) table whether or not they have a match or not, but if they do have a match, returns the matching record on the right table as well. 

![](http://www.khankennels.com/blog/wp-content/uploads/2007/04/left_venn.png)

```
SELECT * FROM TableA
LEFT OUTER JOIN TableB
ON TableA.name = TableB.name

id  name       id    name
--  ----       --    ----
1   Coal      2     Coal
2   Spot      null  null
3   Diamond   4     Diamond
4   Jordi     null  null
```

Hopefully this visualized guide helped a little bit in wrapping your head around how to organize data with SQL. And remember to trust data, not lore. It, in fact, may be the only real thing there is.
