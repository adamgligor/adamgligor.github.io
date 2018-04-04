---
layout: post
title: PowerShell tip - exit code
date: '2018-04-04'
tags: PowerShell
---

A PowerShell script executed by SQL server using `xp_cmdshell` to return a numeric value. 


A PowerShell script executed by SQL server using `xp_cmdshell` returns usually a table so one has to assign the result to a table variable, then query that. 

Suppose that the script is doing some kind of validation and only the outcome (as in true/false) is of interest. Using exit codes it is possible to have the result straight as an int variable in SQL server and not deal with tables. Here's how ...


## The problem

The original version of the script was this: 

```powershell
validate.ps1 

try
{
    # do stuff ... 

    # no errors
    return $false
}
catch [System.Xml.Schema.XmlSchemaValidationException]
{
    # errors
    return $true
}
```

The fragment that executes it from SQL server

```sql 

DECLARE @strCommandValidate VARCHAR(1000)
DECLARE @intError INTEGER

SET @strCommandValidate = 'powershell validate.ps1'
EXEC @intError =  xp_cmdshell @strCommandValidate

SELECT @intError

```

Result will be a table:

![ps](/public/powershell/ps_sql.png)


## Solution 

To have the result straight as an int when executed SQL server one has to return only exit codes from the script

Read on exit codes [here](https://weblogs.asp.net/soever/returning-an-exit-code-from-a-powershell-script)


So with this update the new script is:


```powershell
validate2.ps1 

$validationError = 0 

try
{
    # do stuff ... 
}
catch [System.Xml.Schema.XmlSchemaValidationException]
{
    $validationError = 1
}

EXIT $validationError
```

And the result will be a single int value:

![ps](/public/powershell/ps_sql_int.png)


