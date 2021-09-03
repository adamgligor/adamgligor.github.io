---
layout: post
title: Notes on a database database
description: 
date: '2021-02-01'
tags: programming
---

Notes on an database migration ...

## Scenario


A postgres database serving various applications. Some of the data is accessed remotely for analytics purposes using the foreign data wrapper module. Everything is hosted in AWS. The database server needs to be upgraded, additionally analytics and application should be isolated by providing each a separate copy of the data ideally with little on no downtime.

 
Pic 1. 

On the left side the starting setup. 

On the right side the outcome after the migration.

![placeholder](/public/2021/02/2021-02-01-db-migration1.png "migration1")


Pic 2. The migration highlights. 

On the left side adding a new database and replication using DMS. 

On the right side redirecting apps to the new database and spinning down the old database and the temporary DMS.

![placeholder](/public/2021/02/2021-02-01-db-migration2.png "migration2")

## Links 


https://www.postgresql.org/docs/current/postgres-fdw.html

DMS


