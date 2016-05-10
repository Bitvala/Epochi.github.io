---
layout: post
title: First Commit
description: "Its ON, baby"
headline: "Let's Fire up the Engines"
categories: personal
tags: 
  - blogging
  - jekyll
imagefeature: "mongooses.jpg"
comments: false
mathjax: null
featured: true
published: true
---

I'm currently building reddit-like website with voting and commenting, but with cleaner layout. It should look a bit like facebook news feed, but with scores and two top comments under every post to promote discussion.

I'm going to try and describe how I designed my server and database, to balance website responsiveness and server/database loads.
My goals are:
- Have posts sorted by hot, top or new
- On load check if post is voted on by the user
- Send user votes to server only when client view needs to be updated by server
- Batch all votes in server, then send it to database when an amount is reached or timer runs out