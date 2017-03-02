---
layout: post
title:  Odata 4 service tutorial - part 2
date: '2017-03-02'
tags: odata

---


Here's a continuation of the odata services tutorial started in [part 1](http://adam-gligor.github.io/2017/01/29/Odata-4-services-introduction). Covered topic in part 2: linking entities, applying query manually, funtions, actions and singletons


## Linking entities

This is associating a product to a category. Adding, deleting and updating associations works on the same principle just using different http verbs.

```c#
[ODataRoute("({key})/Products/$ref")]
[HttpPost]
public IHttpActionResult CreateProductLink([FromODataUri] int key, [FromBody] Uri link)
{
    var dbCategory = dbContext.Categories.Include("Products").SingleOrDefault(c => c.Id == key);

    if (dbCategory == null)
        return NotFound();

    var productId = GetProductId(link);

    if (dbCategory.Products.Any(p => p.Id == productId))
        return Ok();

    var dbProduct = dbContext.Products.SingleOrDefault(c => c.Id == productId);
    if (dbProduct == null)
        return NotFound();

    dbCategory.Products.Add(dbProduct);
    dbContext.SaveChanges();

    return Ok();
}
```

And the helper method to extract the product key from the body. The implementation for this will vary based on the odata library version. This code works with **Microsoft.OData.Core version 7.0.0**


```c#
private int GetProductId(Uri link)
{
    var builder = new UriBuilder();
    builder.Host = Request.RequestUri.Host;
    builder.Port = Request.RequestUri.Port;
    builder.Path = ODataConfig.ROUTE_PREFIX;
    var serviceBaseUri = builder.Uri;

    var uriParser = new ODataUriParser(ODataConfig.Model, serviceBaseUri, link);
    var odataPath = uriParser.ParsePath();
    var lastSegment = odataPath.LastSegment as KeySegment;
    var productId = lastSegment.Keys.First().Value.ToString();

    return int.Parse(productId);
}
```

The way to call this endpoint is 

```powershell
$body = @{"@odata.id":"http://localhost:61162/odata/Products(2)"} | ConvertTo-Json
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories(3)/Products/`$ref" -Method Post -ContentType "application/json" -Body $body
```

(Note: the ` sign is just an escape chaaracter required by powershell)


To test that it actually works use:

```powershell
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories(3)?`$expand=Products" -Method Get 
```

And also make sure that the 'GetById' method can handle this call by adding the *EnableQuery()* attribute and returning an *IQueryable* as result.


# Applying query manually 

Executing queries works automatically agains queryable sources. What happens if the backend is not a queryable source, for examplle a soap service. 

.... remove the queryable attribute 
parse the options 
can retain sorting ... 
..disable allowed options 

# Functions and actions 


# Singletons 



## References

- Github source code [link](https://github.com/adam-gligor/OdData4Sample)
- The Odata standard [link](http://docs.oasis-open.org)
- Documentation on using on odata4 with web api [link](http://odata.github.io/WebApi)
