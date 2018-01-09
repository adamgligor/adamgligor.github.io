---
layout: post
title: Phpp hosting in Azure
date: '2018-01-08'
tags: azure
---


Free hosting for **phpbb** forum ? Try microsoft azure with in app mysql server.


Up to 10 apps can be hosted for free in the free tier plan. Whith the recenlty infroduced `mysql-in-app` feature, a mysql database can be hosted alongside a web app. For details head [here](https://blogs.msdn.microsoft.com/appserviceteam/2017/03/06/announcing-general-availability-for-mysql-in-app/)


This opens the possibility to host apps that require a database for free. On such app is the php based `phpbb` forum.


## Web app setup

Start by setting up a new app using the `phpbb on mysql` template, see screen below. 

![placeholder](/public/azure/phpbb1.png "phpbb on mysql")

To use `MySQL in App` look for this setting when creating the app, see screen below. 

![placeholder](/public/azure/phpbb2.png "in app mysql")


## Installation

Access the homepage of the newly created app to continue with the setup. To complete this step, find the the mysql connection information.

With in app mysql the connection information is stored in an environment variable, which you can read with a script as described [here](https://blogs.msdn.microsoft.com/appserviceteam/2016/08/18/announcing-mysql-in-app-preview-for-web-apps/)


Tip: there is a handy editor in azure portal to access and edit the files, look for `App service editor` in the application settings blade.

Here's an example script that dumps out the connection to a web page.


```php
<?php
$connectstr_dbhost = '';
$connectstr_dbname = '';
$connectstr_dbusername = '';
$connectstr_dbpassword = '';

foreach ($_SERVER as $key => $value) {
    if (strpos($key, "MYSQLCONNSTR_localdb") !== 0) {
        continue;
    }
    
    $connectstr_dbhost = preg_replace("/^.*Data Source=(.+?);.*$/", "\\1", $value);
    $connectstr_dbname = preg_replace("/^.*Database=(.+?);.*$/", "\\1", $value);
    $connectstr_dbusername = preg_replace("/^.*User Id=(.+?);.*$/", "\\1", $value);
    $connectstr_dbpassword = preg_replace("/^.*Password=(.+?)$/", "\\1", $value);
}

$link = mysqli_connect($connectstr_dbhost, $connectstr_dbusername, $connectstr_dbpassword,$connectstr_dbname);

if (!$link) {
    echo "Error: Unable to connect to MySQL." . PHP_EOL;
    echo "Debugging errno: " . mysqli_connect_errno() . PHP_EOL;
    echo "Debugging error: " . mysqli_connect_error() . PHP_EOL;
    exit;
}

echo "Success: A proper connection to MySQL was made! The my_db database is great." . PHP_EOL;
echo "<br>";
echo "Host information: " . mysqli_get_host_info($link) . PHP_EOL;
echo "<br>";
echo "u " . $connectstr_dbusername;
echo "<br>";
echo "p " . $connectstr_dbpassword;
echo "<br>";
echo "d " . $connectstr_dbname;

mysqli_close($link);
?>
```

Once the setup completed, the forum should be up and running.

## Tweaks 

I ran into an issue that the login on the forum did not work. To fix that I had to manually delete the `install` folder. 


## Conclusion

Definitely not production grade hosting, but for small scale installation or development it is usable. Best part it's absolutely free. 



