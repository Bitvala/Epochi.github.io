---
layout: post
title: Vote structure for reddit like site on Postgres
description: Vote structure for reddit like site on Postgres
headline: Vote structure for reddit like site on Postgres
tags: postgres
comments: false
mathjax: false
featured: true
published: true
categories:
  - webdevelopment
  - Database
---
I'm currently building reddit-like website with voting and commenting, but with cleaner layout. It should look a bit like facebook news feed, but with scores and two top comments under every post to promote discussion.

I'm going to try and describe how I designed voting mechanism. Although my current build will only use Postgres, in the future I plan to add Redis layer on top for faster performance.


### My functionality goals are:

- On page load show user if post is voted on or saved by the user

### Database Sructure

### Tables 

User has two tables that are affected to votes

	CREATE TABLE user_metadata(
      username text,
      ***
      postscore integer DEFAULT 0,
      commentscore integer DEFAULT 0,
      ***
  	PRIMARY KEY (username)
	);

	CREATE TABLE user_data(
      username text,
      ***
      postvote integer[],
      postsave integer[],
      commentvote integer[],
      commentsave integer[],
      posts integer[],
      ***
      PRIMARY KEY (username)
     )


There's also user_auth table that stores all the secure data.
The basic idea for splitting user_data away from user_metadata is because it will mainly be used for archiving, so when the time comes we will be able to put user_data in slower IOPS databases which are significally cheaper.

And  votes table will have one row per unique pair of post and user.

	CREATE TABLE votes(
      post_id integer,
      username text,
      Votes: jsonb,

      FOREIGN KEY (post_id) REFERENCES post (post_id) ON UPDATE CASCADE ON DELETE CASCADE,
      FOREIGN KEY (username) REFERENCES user_data (username) ON UPDATE CASCADE ON DELETE CASCADE,
      CONSTRAINT vote_pkey PRIMARY KEY (post_id, username)  
      );
_A bit on `CONSTRAINT votepkey PRIMARY KEY`. I will have to do some stress tests to see if (username, postid) or (postid, username) works better._

  
Votes jsonb structure:

	{
    	Vote: bool
        Save: bool
        comment_id: {
        		Vote: bool
                Save: bool
            }
    }

I really thought long and hard about which structure to use for this. I basically came down to two possible solutions:

1. Have a table with separate columns for vote,save,voted comments array,saved comments array.
	Main drawbacks of this is that I'd have to somehow exclude all of the null data from returning to user.
    And I'd have to check for duplicates inside comment arrays, which is not native and pretty tricky with postgres arrays.
    
2. Have jsonb that store vote,save,voted comments array,saved comments array.
	Vote data doesn't need any altering when returning to client.
    Jsonb natively support unique keys.

There are also other ways to store votes, but I really wanted to keep it all in one row, because I will be checking for it every time user loads new posts, this way I'm able to give postgres one possible answer, which will give me query speed.

I went with jsonb because the solution seems much cleaner and without any obvious drawbacks.


### Atomicity 

Now this table should be the second most red table and it also needs to be secure against false votes, by understanding if it has been voted on before. 


