---
layout: post
title: PowerShell tip - exit code
date: '2018-04-04'
tags: PowerShell
---


# PowerShell tip - exit code

A PowerShell script executed by SQL server using `xp_cmdshell` returns usually a table value. 

Suppose that the script is doing some kind of validation and only the outcome (as in true/false) is of interest, using exit codes it is possible to have the result straight as an int in SQL server. Here's how ...


## The problem

The original version of the script is this: 

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

Executing this script from SQL server

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

Returning only exit code from a script it is possible to have the result straight as an int in SQL server.

Read on exit codes [here](https://weblogs.asp.net/soever/returning-an-exit-code-from-a-powershell-script)


Updated script


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

Result will be a single value, assignable to int:

![ps](/public/powershell/ps_sql_int.png)



