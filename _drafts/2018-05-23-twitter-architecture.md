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

* Features
* Architectural approaches

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

### Take-1: Relational database

#### Description

When taking the most direct approach, the obvious thing is to have a set of relational tables: `users`, `tweets`, and `followers`. 
*Users* table is nothing but a storage of active/registered users. The *Followers* table stores a row for each existing subscription of one user to another. This will be required when calculating the feed for each user.
Finally, *Tweets* table contains all tweets with a foreign key specifying the author of each tweet.

![twitter-database-design](assets/twitter-database-design.svg)

#### Upsides &#x1F44D;

The obvious strong side of this approach is its undisputable simplicity. Just one database to manage, several tables, everything is stored in one place. As less overhead as possible. To be fair, many start-ups rely on this exact approach when buidling their product at the very beginning. Tumblr is a good example of that.

#### Downsides &#x1F44E;

Well, while being the easiest to implement, this system is not very scalable. Of course, it can handle a hundred users. If everyone would subscribe to everyone the `followers` table would grow to be 10,000 rows, which is fine unless the user base starts to really grow.
You could imagine that the `select` statement to calculate the timeline of each user would eventually take forever to complete or would hit a memory boundary.

### Take-2: Sharded relational database

#### Description
#### Upsides &#x1F44D;
#### Downsides &#x1F44E;

### Take-3: Pre-calculated feeds

#### Description
#### Upsides &#x1F44D;
#### Downsides &#x1F44E;
