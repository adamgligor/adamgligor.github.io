---
draft: true
---

Tips for creating better powershell cmdlets from c#. 


... 

# The `-Verbose` switch


Write-Verbose(...) only prints when flag is set 


# The `-WhatIf` switch 

```
if (ShouldProcess(deletion.Topic, deletion.Action)) ...
```


# Showing progress bar 

```
WriteProgress ...
```



<!-- ## Loading and unloading a dll containging cmdlets 

Use the `Import-Module` -->


