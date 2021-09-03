---
layout: post
title: Notes on a database database
description: 
date: '2021-02-01'
tags: programming
---

Notes on an database migration ...

## Scenario


A postgres database serving various applications. The data is also accessed remotely for analytics purposes using the foreign data wrapper module. Everything is hosted in AWS and the database is of managed type. 

The database server needs to be upgraded, additionally analytics and application should be isolated by providing each a separate copy of the data. The migration should ideally be done with with little or no downtime.

 
*Pic 1. The setup*

On the left side the starting setup. 

On the right side the outcome after the migration.

![placeholder](/public/2021/02/2021-02-01-db-migration1.png "migration1")


Postgres has good native tools to copy backup and restore data and good replication support but for this the AWS Database Migration Service was used. This is a managed service that can do one time copy and or ongoing replication between a source and target and supports many of the common database systems. 

The steps of the migration.

1) Spin up a new database with latest postgres engine.
2) Set up a temporary dms replication from the existing to the new app database.
3) Set up a permanent dms replication from the new app database to the bi database.
4) Once the data transfer is ready switch the applications to the new database.
5) Shut down the old database and temporary replication.


*Pic 2. Migration visualized*

On the left side adding a new database and replication using DMS. 

On the right side redirecting apps to the new database and spinning down the old database and the temporary DMS.

![placeholder](/public/2021/02/2021-02-01-db-migration2.png "migration2")

The DMS has a great deal of flexibility but also many limitation since it tries to be generic and cover many common use cases. Data types might not be fully supported like specialized data types used by the postgis geo spatial extension, binary data can be problematic or slow to transfer and there's a lot of fiddling involved in the initial setup. 

Despite all this it's a great tool that can help achieve zero downtime migrations.

## Links 

- The fwd [here](https://www.postgresql.org/docs/current/postgres-fdw.html)
- the AWS DMS [here](https://aws.amazon.com/dms/)


