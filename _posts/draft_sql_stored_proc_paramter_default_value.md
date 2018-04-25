---
draft: true
---

https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/specify-parameters?view=sql-server-2017

```
CREATE PROCEDURE [dbo].[proc_foo]
	  @first int = 1,
	  @second varchar(10)  = 'foo'
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

    -- Insert statements for procedure here
	SELECT @first,@second
END
```

```
DECLARE @RC int
DECLARE @first int
DECLARE @second varchar(10)



EXECUTE @RC = [dbo].[proc_foo]  -- uses defaults

EXECUTE @RC = [dbo].[proc_foo]  @first ,@second -- nulls

EXECUTE @RC = [dbo].[proc_foo] 2, 'ZZZ' - ok , uses supplied constants 

SET @first = 2
SET @second = 'ZZZ'
EXECUTE @RC = [dbo].[proc_foo]  @first ,@second -- ok, uses supplied variables


SET @second = 'ZZZ'
EXECUTE @RC = [dbo].[proc_foo]  @second -- not ok , dos not seem to be possible to supply sublset of paramters !








```


