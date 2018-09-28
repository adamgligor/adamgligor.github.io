---
layout: post
title: Mssql stored procs and arguments
date: '2018-05-01'
tags: Sql
---

The correct way of supplying arguments to an msqql stored procedure.


Sql management studio generate scripts can be slightly missleading, for example the script generated to execute a stored procedure. This is a reminder for myself to not make this same mistake again.

Here's my dummy stored procedure with default values for the arguments:

```sql
CREATE PROCEDURE [dbo].[proc_foo]
	@first int = 1,	
	@second varchar(10)  = 'foo'
AS
BEGIN
	SELECT @first,@second
END
```

So I want to call this proc by supplying the first argument and using the default for the second. 

From sql management studio, right click the stored proc and choose `script stored procedure as > create to > new query window`. 

The generated script is this: 

```sql
DECLARE @RC INT
DECLARE @first INT
DECLARE @second VARCHAR(10)

EXECUTE @RC = [dbo].[proc_foo]  @first, @second 

```

then my instinct was to init the variablelike this `SET @first = 10` before the `EXECUTE` call, expecting the return to be `10,foo` ... right ? Wrong ! 

The correct syntax to supply the arguments is either inline or using the syntax @name = @value

Example: 

```sql
DECLARE @RC INT
DECLARE @first VARCHAR(10)
DECLARE @second VARCHAR(10)

-- inline
EXECUTE @RC = [dbo].[proc_foo] 2, 'bar' --returns 2,bar

-- or with variables
SET @first = 10
SET @second = 'bar'
EXECUTE @RC = [dbo].[proc_foo]  @first= @first, @second = @second --returns 10,bar  

-- or with just one (and the other takes the default value)
SET @first = 10
EXECUTE @RC = [dbo].[proc_foo]  @first = @first  --returns 10,foo  

SET @second = 'bar'
EXECUTE @RC = [dbo].[proc_foo]  @second = @second  --returns 1,bar

```

More on stored proc parameters on msdn [here](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/execute-a-stored-procedure?view=sql-server-2017)

