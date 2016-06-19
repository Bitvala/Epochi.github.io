---
layout: post
title: '''Hot'' Voting with React, Node and Postgres'
description: '''Hot'' Voting with React, Node and Mongoose'
headline: '''Hot'' Voting with React, Node and Mongoose'
tags: 'react,node,node.js,mongodb,mongoose'
comments: false
mathjax: false
featured: true
published: true
categories:
  - webdevelopment
---
I'm currently building reddit-like website with voting and commenting, but with cleaner layout. It should look a bit like facebook news feed, but with scores and two top comments under every post to promote discussion.

I'm going to try and describe how I designed my server and database. Although my current build will only use Postgres, in the future I plan to add Redis layer on top for faster performance.

### Table of Contents

1. Functionality Goals
2. Database
	2.1. Database Structure
    2.2. Tables


### My functionality goals are:

- Have posts sorted by hot, top or new
- On page load show user if post is voted on or saved by the user
- Let user connect with local authentication or Google, Facebook


### Database Sructure

### Tables 

User has username which is just lowercase name, so there are no users with extremely similair names. 
I'm putting user authentication to a separate table, because of security concerns. This way I can make sure user_auth row info never get out from my server.

CREATE TABLE user_data(
  username text,
  name text NOT NULL,
  email text UNIQUE,
  postscore integer DEFAULT 0,
  commentscore integer DEFAULT 0,
  postvote integer[],
  postsave integer[],
  commentvote integer[],
  commentsave integer[],
  posts integer[],
  data json,
  PRIMARY KEY (username)
);
  
CREATE TABLE user_auth(
  username text,
  email text,
  hashed_password text,
  salt text,
  authToken text,
  thirdPartyAuth json,
  PRIMARY KEY (username),
  FOREIGN KEY (username) REFERENCES user_data (username) ON UPDATE CASCADE ON DELETE CASCADE,
  FOREIGN KEY (email) REFERENCES user_data (email) ON UPDATE CASCADE
);  


CREATE OR REPLACE FUNCTION user_lowercase()
  RETURNS "trigger" AS
$BODY$
BEGIN
	new.email := lower(new.email);
	new.username := lower(new.username);
	return new;
END
$BODY$
  LANGUAGE 'plpgsql' VOLATILE;

CREATE TRIGGER user_lower_username
  BEFORE INSERT OR UPDATE
  ON users
  FOR EACH ROW
  EXECUTE PROCEDURE user_lower_username();





CREATE TABLE "session" (
  "sid" varchar NOT NULL COLLATE "default",
        "sess" json NOT NULL,
        "expire" timestamp(6) NOT NULL
)

WITH (OIDS=FALSE);
ALTER TABLE "session" ADD CONSTRAINT "session_pkey" PRIMARY KEY ("sid") NOT DEFERRABLE INITIALLY IMMEDIATE;

yp0=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO users;

CREATE INDEX articles_published_at_index ON articles(published_at DESC NULLS LAST);
CREATE INDEX CONCURRENTLY articles_published_at_index ON articles(published_at DESC NULLS LAST);

CREATE TABLE post(
 post_id SERIAL PRIMARY KEY ,
 score integer DEFAULT 0,
 kind smallint,
 voteup integer DEFAULT 0,
 votedown integer DEFAULT 0,
 author text NOT NULL,
 date timestamp DEFAULT current_timestamp, 
 title text NOT NULL,
 subport text NOT NULL,
 tags text[],
 commentcount integer DEFAULT 0,
 data json
);

CREATE INDEX CONCURRENTLY post_top_index ON post(voteup DESC NULLS LAST);
CREATE INDEX CONCURRENTLY post_new_index ON post(date DESC);

CREATE INDEX CONCURRENTLY post_hot_index ON post(date DESC NULLS LAST);


CREATE TABLE post_vote(
  post_id integer,
  user_vote smallint,
  username text,
  FOREIGN KEY (post_id) REFERENCES post (post_id) ON UPDATE CASCADE ON DELETE CASCADE,
  FOREIGN KEY (username) REFERENCES user_data (username) ON UPDATE CASCADE ON DELETE CASCADE,
  CONSTRAINT vote_pkey PRIMARY KEY (post_id, username)  
);

CREATE TABLE comment(
  post_id integer,
  parent_id integer,
  comment_id integer,
  score integer DEFAULT 0,
  voteup integer DEFAULT 0,
  votedown integer DEFAULT 0,
  date timestamp DEFAULT current_timestamp,
  data json,
  PRIMARY KEY (post_id, parent_id, comment_id),
  FOREIGN KEY (post_id) REFERENCES post (post_id)  ON UPDATE CASCADE ON DELETE CASCADE
);

CREATE TABLE comment_vote(
  post_id integer,
  user_vote smallint,
  username text,
  FOREIGN KEY (post_id) REFERENCES post (post_id) ON UPDATE CASCADE ON DELETE CASCADE,
  FOREIGN KEY (username) REFERENCES user_data (username) ON UPDATE CASCADE ON DELETE CASCADE,
  CONSTRAINT comment_vote_pkey PRIMARY KEY (post_id, username)  
);

/*
data json: 
  kind text (link, textpost, image)
  bodytext text[],
  imageurl text,
  nsfw bool,
  top_comments json,
  edited date,
  mod_reports text[],
  user_reports text[],
*/
















