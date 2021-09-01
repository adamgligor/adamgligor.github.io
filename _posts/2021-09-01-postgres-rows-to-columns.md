---
layout: post
title: Rows to columns using sub-queries in postgres
description: 
date: '2021-09-01'
tags: programming
---


Converting rows to columns using sub-queries in postgres. 


## Scenario 

Given two entities `users` and `roles`, each entity has an identifier and a name and users can have one or more roles (say up to 3). 

How can this be modeled in postgres then queried to produce a list of users and associated roles. 

Example. 

Input: 
  - two roles reader and writer
  - two users John and Mary. John is reader and Mary is reader and writer.

Output: 
  - John, reader
  - Mary, reader, writer

## Option 1 

First the obvious, using many to many relationship. Ths involves setting up three tables 
```
CREATE TABLE public.users( id integer, name character varying)
CREATE TABLE public.roles ( id integer, name character varying)
CREATE TABLE public.user_roles ( user_id integer, role_id integer)
```
and the query 
```
    SELECT * 
    FROM users 
    JOIN user_roles ON user.id = user_role.user_id
    JOIN roles ON role.id = user_role.role_id
```
which needs one further aggregation to get one row per user.

## Option 2

Postgres supports array type columns and related sub-queries in the select clause, so this is also possible. 


The db model.
```
CREATE TABLE public.users (id integer, name character varying, role_ids integer[], CONSTRAINT pk_users PRIMARY KEY (id))

CREATE TABLE public.roles (id integer, name character varying, CONSTRAINT pk_roles PRIMARY KEY (id))
```
fill some test data in. 
```
INSERT INTO roles (id, name) SELECT generate_series(1,5000), 'role' || md5(random()::text)

INSERT INTO users (id, name, role_ids) 
SELECT *, 'role' || md5(random()::text), ARRAY[floor(1+ random() * 1000), floor(1+ random() * 1000)]  FROM generate_series(1,5000)
```
Then the query making use of a sub-query in the select clause. This time the result is already in the expected format, each user is one row and the roles names are also included.
```
    SELECT
    *, (SELECT ARRAY(SELECT name FROM roles WHERE id = ANY(u.role_ids))) AS "role_names"
    FROM users AS u WHERE id IN (1,10,100,1000)
```
which returns role_names as array *-or-*
```
    SELECT
    *, (SELECT string_agg(name, ',') FROM roles WHERE id = ANY(u.role_ids)) as "role_names"
    FROM users AS u WHERE id IN (1,10,100,1000)
```
which returns role_names as a concatenated string.


The execution plan for this query looks as follows
```
1.Index Scan using pk_users on public.users as u (...)
  Index Cond: (u.id = ANY ('{1,10,100,1000}'::integer[]))
  2.Result (...)
    3.Index Scan using pk_roles on public.roles as roles (...)
      Index Cond: (roles.id = ANY (u.role_ids))
```
It requires two index seeks to fulfill.