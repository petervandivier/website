+++ 
draft = false
date = 2024-11-15T16:50:31-05:00
title = "SQL Server vs. PostgreSQL - which is better?"
description = "The very hottest of takes."
slug = "sql-server-vs-postgres"
authors = ["Peter Vandivier"]
tags = ["sql-server","postgresql"]
categories = []
externalLink = ""
series = []
+++

In the long and varied public discourse comparing SQL Server to PostgreSQL, no one has ever publicly declared which is better. It's time that changed and I'm the guy to do it.

> Is SQL Server worth the sticker price?

Obviously the only possible analogy to be made here is that a database platform is like a kitchen. To create the best food, you must first have the finest cutlery, restaurant grade appliances, only the most artisanal of farm-to-table ingredients. This is a complete inclusive list of all the things you must have in order to create the best of all possible foods.

You should probably also take like... one cooking class or something. But that's all. Your kitchen is now ready to be evaluated for a Michelin star.

Except you have this neighbor who used to be a line chef at a diner. It's well known in the HOA that your neighbor cooks better food than you. Obviously this is due to politics. Neighbor is _poor_ and they use strange _ethnic_ utensils in their kitchen. 

> Wait - I don't get it. Is this humorous misdirection?

SQL Server is the expensive cookware.

The platform you build on has less to do with success than your ability to use it. If you're bad at cooking, the food is gonna suck no matter what you spent on pots and pans. Carry the analogy forward though. You should probably know how to use a carving knife before you try to dress a bird; you might not want to deep-fry something for the first time if you [watched one youtube video](https://youtu.be/fs0KLgNzQHA) on how to do it.

> It's a lot easier to talk about why something sucks than how to make it good. Answer the question.

Okay, fair. SQL Server is "better" most of the time. But it won't make your app good if you suck at databaseing. 

Likewise PostgreSQL won't make your app suck if you're good at databaseing. 

All other things being equal, I choose PostgreSQL for greenfield development. If you've been in my kitchen this won't surprise you - I cook quite a lot on very cheap cookware. 

> So PostgreSQL is better. Why would anyone use SQL Server?

If you've ever worked in food service, you probably weren't in charge of what range was bought or what supplier installed the walk-in. High-end kitchens exist, so does high-end cookware. Just 'cause you wouldn't (or couldn't) buy it yourself for your home doesn't mean you turn your nose up at it in an industrial setting. 

I'm an okay cook and I still go out to nice restaurants. I fancy myself a pretty good database developer in my home lab and I still love working on the Microsoft stack. 

----

There exist top-tier teams in bleeding-edge scenarios where one platform is better than the other. SQL Server wins out a lot of the time in the true top 1% of stacks for the same reason that high-end restaurants pay their staff so well relative to other joints: _because_ the platform doesn't make the product, the team does. The platform can however help the team get _just that last little bit of value out_ (if you're willing to 10x your spend for that last 1%). This matters at the highest levels - but that's the _only_ place it matters. If you're reading this deep into my blog post about it, you're not there.

----

A lot of people fancy themselves much better at cooking than they are. A lot of people talk a big game about their tech stack. I don't like to tear down folks for being proud of a project. But when asked, I usually advise that your stack is probably much more _normal_ than you may have previously pictured. Given that assumption - just... use whatever you want to, buddy.

Switch from one to the other if you like. Switch back. Run both concurrently with a bespoke cross-platform replication setup. But don't do it 'cause I (or some other internet rando) told you to.
