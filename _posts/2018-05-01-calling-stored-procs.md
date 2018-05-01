---
layout: post
title: Calling stored procs
date: '2018-05-01'
draft: true
tags: Sql
---

The correct way of supplying arguments to a stored procedure.


Don't count on sql management studio scripts to generate stored procedure calls correctly. 

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

With sql management studio, I choose `script stored procedure to new query window` and get this script: 

```sql
DECLARE @RC INT
DECLARE @first INT
DECLARE @second VARCHAR(10)

EXECUTE @RC = [dbo].[proc_foo]  @first, @second 
```

then add

```
SET @first = 10
```

before `EXECUTE` and expecting it to return `10,foo` ... right ? Wrong ! 


The correct syntax to supply the arguments is this: 

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
```

So to supply ony some of the parametrers

```sql
DECLARE @RC INT
DECLARE @first VARCHAR(10)
DECLARE @second VARCHAR(10)


SET @first = 10
EXECUTE @RC = [dbo].[proc_foo]  @first = @first  --returns 10,foo  

SET @second = 'bar'
EXECUTE @RC = [dbo].[proc_foo]  @second = @second  --returns 1,bar

```

Takeaway, aleays specify parameters inline or using the syntax @name = @value

More on sproc parameters from msdn [here](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/execute-a-stored-procedure?view=sql-server-2017)

