---
layout: post
title:  "Trust Data, Not Lore: SQL Joins Visualized"
date:   2017-08-30 19:59:13 -0400
---

![](https://68.media.tumblr.com/52c04c960042f7e614e36fce0833f995/tumblr_oow3quxpdg1swxsyto2_500.jpg)
> Sound advice from Star Trek The Next Generation.

Quoting a very cool [YouTube video](https://www.youtube.com/watch?v=yWO-cvGETRQ) by Kurazgesagt, *"When it comes down to it, information is nothing tangible. It's typically understood as the property of the arrangement of particles."* 

What this means is, a bunch of carbon atoms organized one way can create coal. The very same carbon atoms arranged another way can create diamonds. Zoomed in, the atoms are the same. What changes is the information. Our mental representation of physical things (e.g. a diamond or coal) is also information, and the process by which these *thoughts* manifest in the first place is also, in the end, different neuronal activity in the brain, which, once again, are just the arrangement of particles, by our definition, also information. 

In a way, the whole universe is just a collection of information, and how we perceive ourselves in it, is a product of our ability to see the *context*, or interrelatedness, or grouping, of said information. In this frame of reference, zoomed out, all of the ideas, philosophies, friendships, technology, and concepts that exist in this blip of space-time on planet Earth, and even Earth itself, can be seen as just different information, interrelated, arranged, and interpreted, in different ways (The carbon particles in any configuration are objectively just that, carbon particles, but to be conscious is to know that we, as humans, tend to burn one configuration and wear the other in a ring). 

![](http://www.nndb.com/people/934/000023865/claude-shannon-1-sized.jpg)

> "I just wondered how things related to one another." - Claude Shannon, father of information theory

One philosophical view is that it's all just information out there, and SQL (Structured Query Language) gives us a way to organize and see connections between different tables or groupings of information. Information that is orderly and is able to be referenced and analyzed in relation to each other is useful, and can be referred to as data. When we zoom back in, our job as developers is to be masters of information and data. We create *context* and map *relationships* out of information in order to bring function and order to our software applications. This is why SQL queries and joins are important: to make sense of how information relates to each other.

Many novice programmers, including myself starting out, get confused when first learning SQL and how exactly to query specific information from tables in a database. Tables represent sets of information about specific domains, and the results of 'join queries' in SQL represent the interrelatedness of elements between these domains, but these can be hard to conceptualize.

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

The following are the three frequent 'join queries' one can use in SQL to query different matching records from each table, visually explained.

An **inner join** query only returns records with matches in both tables. 

![](http://www.khankennels.com/blog/wp-content/uploads/2007/04/venn1.png)

```
SELECT * FROM TableA
INNER JOIN TableB
ON TableA.name = TableB.name
id  name       id   name
-- ----        --  ----
1   Coal       2    Coal

3   Diamond    4    Diamond

```

An **outer join** query returns all the records from T1 and T2, with matching records from both sides where available.

![](http://www.khankennels.com/blog/wp-content/uploads/2007/04/outervenn.png)

```
SELECT * FROM TableA
FULL OUTER JOIN TableB
ON TableA.name = TableB.name
id    name       id    name
--    ----       --    ----
1     Coal       2     Coal

2     Spot       null  null

3     Diamond    4     Diamond

4     Jordi      null  null

null  null       1     Picard

null  null       3     Neuron
```

A **left join** query returns all of the records in the first (left) table whether or not they have a match or not, but if they do have a match, returns the matching record on the right table as well. 

![](http://www.khankennels.com/blog/wp-content/uploads/2007/04/left_venn.png)

```
SELECT * FROM TableA
LEFT OUTER JOIN TableB
ON TableA.name = TableB.name

id  name       id    name
--  ----       --    ----
1   Coal       2     Coal
2   Spot       null  null
3   Diamond    4     Diamond
4   Jordi      null  null
```

Repeatedly visualizing each SQL join I had to make helped clarify them for me when I was learning. 

When organized properly, we can trust data, and its relationship to other data gives us context and a narrative. Organizing data helps us make sense of a complicated universe, whether in a computer program or in reality, which is, perhaps in and of itself, just information. 
