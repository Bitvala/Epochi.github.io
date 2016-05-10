---
layout: post
title: "'Hot' Voting with React, Node and Mongoose"
description: "'Hot' Voting with React, Node and Mongoose"
headline: "'Hot' Voting with React, Node and Mongoose"
categories: 
  - personal
tags: "react,node,node.js,mongodb,mongoose"
imagefeature: mongooses.jpg
comments: false
mathjax: false
featured: true
published: true
modified: ""
---
I'm currently building reddit-like website with voting and commenting, but with cleaner layout. It should look a bit like facebook news feed, but with scores and two top comments under every post to promote discussion.

I'm going to try and describe how I designed my server and database, to balance website responsiveness and server/database loads.

My goals are:

- Have posts sorted by hot, top or new
- On load check if post is voted on by the user
- Send user votes to server only when client view needs to be updated by server
- Batch all votes in server, then send it to database when an amount is reached or timer runs out
