---
title:  "Designing Twitter"
date:   2018-05-22 22:00:00 +0200
keywords: howto python benchmark fasters directory size os walk scandir listdir
description: "Review of approaches to designing a system like Twitter, Tumblr, or Instagram"
image: https://image.ibb.co/jdwRhS/vim_python_pipenv.jpg
---

## Summary

This article describes different approaches to designing a system like Twitter.

## Table of contents

* <a href="#features">Features</a>
* <a href="#architectural-approaches">Architectural approaches</a>
  * <a href="#relational-database">Relational database</a>
  * <a href="#sharded-relational-database">Sharded relational database</a>
  * <a href="#pre-calculated-feeds">Pre-calculated feeds</a>

<!--more-->

## Features

First of all, let's specify a list of features that will be present in the system.
Typically, a system like Twitter consists of user accounts that can follow each other and something called feed/dashboard, or timeline, a place where all posts made by user's follow-pull are gathered and rendered in a reversed time manner.

* Tweet

  Each user is able to tweet or post a simple textual message. For the sake of simplicity we don't consider retweets, likes, or comments.

* Follow

  Any user can follow any other user. When user follows someone else a subscription is created. Whenever followed user is posting something, the follower will see these tweets in his feed.

* Feed

  Collection of tweets made by followed users organized in a time-reversed fashion. 
  An important addition to mention is that a system like twitter prefers eventual consistence over immediate conistency. It is better to get fast read rather than immediate write.

## Architectural approaches

We will review several approaches, starting with the simplest one, like storing everything in a relational database. But eventually the system is going to target millions of users making hundreds of thousands tweets a day.

<a name="relational-database">
### Take-1: Relational database 

#### Description

When taking the most direct approach, the obvious thing is to have a set of relational tables: `users`, `tweets`, and `followers`. 
*Users* table is nothing but a storage of active/registered users. The *Followers* table stores a row for each existing subscription of one user to another. This will be required when calculating the feed for each user.
Finally, *Tweets* table contains all tweets with a foreign key specifying the author of each tweet.

![twitter-database-design](/assets/twitter-database-design.svg)

#### Upsides &#x1F44D;

The obvious strong side of this approach is its undisputable simplicity. Just one database to manage, several tables, everything is stored in one place. As less overhead as possible. To be fair, many start-ups rely on this exact approach when buidling their product at the very beginning. Tumblr is a good example of that.

#### Downsides &#x1F44E;

Well, while being the easiest to implement, this system is not very scalable. Of course, it can handle a hundred users. If everyone would subscribe to everyone the `followers` table would grow to be 10,000 rows, which is fine unless the user base starts to really grow.
You could imagine that the `select` statement to calculate the timeline of each user would eventually take forever to complete or would hit a memory boundary.

#### Technologies

* Database: **MySQL/PostgreSQL**
* Backend: **Python (Django)/Ruby on Rails/PHP** 

<a name="sharded-relational-database">
### Take-2: Sharded relational database

#### Description

A different approach could be evolved out of an original single database design. It is possible to significantly improve scalability of the database in this particlular case by shadring actual tweet storage and indexes.
Since the feed is based on chronologically ordered collection of tweets, the sharding might be done on a time basis. 
Separating database indexes from the actual storage is an attempt to speed up query execution.
There would still be one master in each of the sections. However, by introducing slave replicas and shards we can significantly improve response times.

![twitter-sharded-design](/assets/twitter-sharded-design.svg)

#### Upsides &#x1F44D;

Indexes are now stored in separate shards. The actual calculation of the timeline will be performed only by scanning index shards. 

#### Downsides &#x1F44E;

A design like this would eventually hit the read operation ceiling when talking to index shards. 

#### Technologies

* Database: **MySQL** *(no PostgreSQL due to the lack of decent logical replication)*
* Backend: **Same**

<a name="pre-calculated-feeds">
### Take-3: Pre-calculated feeds

#### Description

If we bring the idea of trading off write speeds for read speed to its absolute, we sould eventually take a look at the approach that pre-calculates the feeds (also called materialized timelines). The immediate consistency of the write operation is traded off for a faster feed read operation for each user.
In order to implement this, each time someone posts a new tweet the timeline of each user must be updated.
It seems obvious that some caching solution is in order here.
Memcached might be an option, but it stores the data in a `blob` format. Binary *blob* format will require us to pull out the whole timeline of a particluar user, append a new tweet to it, and then put it pack to the cache.

Contrary, Redis has a notion of list data structure, which allows us to append tweets without extracting whole timeline from the cache.
With this approach, each time some user requests a feed, a pre-calculated ready-to-read feed will be obtained from the Redis.

To make things even better, it would be nice to introduce a **load balancer** which has been deliberately ignored in the previous considerations. A load balancer would choose a proper Redis instance to perform a read or write operation.

The backend application would request a list of followers whenever some user posts a new tweet. Based on this list the backed will add new tweet to all of the required lists in Redis.

![twitter-materialized-view](/assets/twitter-materialized-view.svg)

#### Upsides &#x1F44D;

The time complexity of read operation is *O(1)*. Write complexity, however, is *O(n)*. We must append a new tweet to every timeline that belongs to a follower.

#### Downsides &#x1F44E;

The amount of stored data might become quite great in size. Redis will require a replication in order to keep serving requests even when one intance with pre-calculated feed dies.
Since Redis is an in-memory database, each instance would also need a great amount of RAM to operate properly, which is an obivous price we pay for an instant access to user's timeline.

Another dangerous case is presented by the users with millions of followers. Any update they post to their followers must be written to millions of Redis lists for pre-calculated timelines.
When taking replicas into account, this could take an unpredictable amount of time.

A possible solution to that is to keep the tweets of popular users somewhere separate and avoid the whole on-write timeline update story completely. 
Instead, whenever any of their followers requests a tweet feed, the backend app will manually merge the tweets of the popular user inside the timeline. 
Doing this in runtime during read operation is obviously not a *O(1)* complexity. However, this prevents us from generating a write avalance each time they post something.

#### Technologies

* Database: **Redis (cached feed), MySQL (followers table)**
* Backend: **Same**
