---
layout: post
title:  Odata 4 service tutorial - part 2
date: '2017-03-02'
tags: odata
---


This is the second part of the odata services tutorial. First part can be found [here](http://adam-gligor.github.io/2017/01/29/Odata-4-services-introduction). Following topics will be covered: linking entities, applying query manually, functions, actions and singletons. 

All code samples are built for the same data model as in part one, that is categories and products, products belog to a category.


## Linking entities

Linking entities can be done via a separate http request. This is the code for linking a product to a category. Adding, deleting and updating links works on the same principle just using different http verbs.

```c#
[ODataRoute("({key})/Products/$ref")]
[HttpPost]
public IHttpActionResult CreateProductLink([FromODataUri] int key, [FromBody] Uri link)
{
    var dbCategory = dbContext.Categories.Include("Products")
        .SingleOrDefault(c => c.Id == key);

    if (dbCategory == null)
        return NotFound();

    var productId = GetProductId(link);

    if (dbCategory.Products.Any(p => p.Id == productId))
        return Ok();

    var dbProduct = dbContext.Products
        .SingleOrDefault(c => c.Id == productId);

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


And a query to list a category together with all the products, to test that the association worked:

```powershell
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories(3)?`$expand=Products" -Method Get 
```

And also make sure that the 'Category/GetById' method has the *EnableQuery()* attribute and it is returning an *IQueryable* as result.


# Applying query manually 

Executing queries works automatically agains queryable sources. If the backend is not a queryable source, for examplle a soap service the query can be parsed and applied manually. 


First remove the *queryable* attribute from the controller action, add a *ODataQueryOptions<T>* parameter to retrive the query options as they are received, 
parse the query options and apply them as fit. 


Bits of the query option (like filtering, sorting, paging) can be applied individually using the *option.ApplyTo(Iqueryable)*, it might make sense for example to pass the filter options to the downstream service and apply filtering afterwards at the odata service level.

There's an article that describes how to parse the ODataQueryOptions [link](https://blogs.msdn.microsoft.com/alexj/2012/12/06/parsing-filter-and-orderby-using-the-odatauriparser/)


# Functions and actions 

Functions and actions are useful for supporting functionality outside of CRUD operations, like queries and commands. There are three types based on how they relate to entities: 

- Functions/actions bound to a collection
- Functions/actions bound to an entity
- Unbound functions/actions (not bound to any collection or entity)


Functions or actions first have to be declared in the edm model. Here are a few examples. Excuse for the unispired names and dumb implementations.

```c#

private static IEdmModel GetEdmModel()
{
    var builder = new ODataConventionModelBuilder();
    ...

    //function bound to collection, returns single entity 
    var func1 = builder.EntityType<Category>().Collection
        .Function("ACollectionBoundFunction");
    func1.Parameter<int>("aParam");
    func1.ReturnsFromEntitySet<Category>("Categories");
    func1.Namespace = "Sample.Functions";

    //function bound to entity, returns primitive
    var func2 = builder.EntityType<Category>().Function("AnEntityBoundFunction");
    func2.Returns<int>();
    func2.Namespace = "Sample.Functions";

    //unbound function, returns collection  
    var func3 = builder.Function("UnboundFunction");
    func3.ReturnsCollectionFromEntitySet<Category>("Categories");

    //action bound to a an entity
    var rateAction = builder.EntityType<Product>().Action("Rate");
    rateAction.Returns<bool>();
    rateAction.Parameter<int>("rating");
    rateAction.Namespace = "Sample.Actions";

    ...
}
```

Then the methods are implemented in the controller. The functions related to categories go into the categories controller. This is not a requirement just makes sense to have it so.

```c#

[ODataRoutePrefix("Categories")]
public class CategoriesController : ODataController
{
    ProductsDbContext dbContext = new ProductsDbContext();
    
    ...

    [ODataRoute("Sample.Functions.ACollectionBoundFunction(aParam={aParam})")]
    [HttpGet]
    public IHttpActionResult ACollectionBoundFunction(int aParam)
    {
        return Ok(dbContext.Categories.FirstOrDefault());
    }

    [ODataRoute("({key})/Sample.Functions.AnEntityBoundFunction")]
    [HttpGet]
    public IHttpActionResult AnEntityBoundFunction([FromODataUri] int key)
    {
        return Ok(key);
    }
    ...
} 

```
The unbound function gets its own controller since it's unrelated to any entities.

```c#

public class FunctionsController : ODataController
{
    ProductsDbContext dbContext = new ProductsDbContext();

    [ODataRoute("UnboundFunction")]
    [HttpGet]
    public IHttpActionResult UnBoundFunction()
    {
        return Ok(dbContext.Categories.Take(3));
    }
}
``` 

Finally these will be awailabe at the following urls: 


```powershell
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories/Sample.Functions.ACollectionBoundFunction/" -Method Get 

Invoke-WebRequest -URI "http://localhost:61162/odata/Categories(3)/Sample.Functions.AnEntityBoundFunction/" -Method Get 

Invoke-WebRequest -URI "http://localhost:61162/odata/UnboundFunction" -Method Get 
```

**Note** The tailing slashes seem to be important here! Might not work without them.


# Singletons 


Singletons are a way to name special entities. In case there is only one entity of a certain kind it replaces the need for a songle entity collection.

Singletons are defined in the edm model as follows:  


```c#

private static IEdmModel GetEdmModel()
{
    var builder = new ODataConventionModelBuilder();
    ...
	//a singleton
    var singleton = builder.Singleton<Category>("TheOne");
    ...
}
```

Then the controller to implement the method: 

```c#

public class SingletonsController : ODataController
{
    ProductsDbContext dbContext = new ProductsDbContext();

    [ODataRoute("TheOne")]
    [HttpGet]
    public IHttpActionResult TheOne()
    {
        return Ok(dbContext.Categories.First());
    }
}
```

And to invoke it use:

```powershell
Invoke-WebRequest -URI "http://localhost:61162/odata/TheOne/" -Method Get 

```

## References

- Github source code [link](https://github.com/adam-gligor/OdData4Sample)

