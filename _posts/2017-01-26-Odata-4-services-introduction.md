---
layout: post
title:  Odata 4 service tutorial - part 1
date: '2017-01-29'
tags: programming
---


Geting started with OData 4 services ? I switched to a new project in the beginning of this year so I had to pick up a bit of odata knowledge. I'll cover the basics in the following articles. 

## What is odata

**Update** 2017-02-05 - Added powershell command examples for invoking the endpoints  

OData (Open Data Protocol) is an OASIS standard that defines a set of best practices for building and consuming RESTful APIs. 
OData helps you focus on your business logic while building RESTful APIs without having to worry about the various approaches to define request and response headers, status codes, HTTP methods, URL conventions, media types, payload formats, query options, etc. 
OData also provides guidance for tracking changes, defining functions/actions for reusable procedures, and sending asynchronous/batch requests.

Source: [odata.org](http://www.odata.org/)

## Prerequisites

For the purpuse of ilustrating basic capabilities of an odata service and some of the implementation details, I created a demo project. The complet source code can be found on github, see references section.

The data model is very simple and consists of two entities **categories**, and **products**, with one to many relationship between them.

Category: Id, Name, Products

Product: Id, Name, Price, Cateogory

The example code will cover the following topics: 

 - Basic crud operations
 - Navigation and entity references
 - Queries 

The code was built with: visual studio 2015, .net 4.6.2, and odata 4, web api 2.2, entity framework 6 and sql local db.

## Initial setup and gotchas

To start with odata 4 service creation install the  Microsoft.AspNet.Odata nuget package. 

There is one gotcha I encountered with this, after pulling in all the dependencis the solution would not compile. The error message said something like: 

*Error:  Multiple assemblies with equivalent identity have been imported: ... remove one of the duplicate references*

The fix for me was updating all packages to the lastest version.

For data persistency, install and configure entity framework 6, this is beyond the scope of this article.


## Odata Edm model 

To initialize the odata service call *MapODataServiceRoute* extension on a HttpConfiguration instance, this requires an entit data model to infer knowledge about the data model.
The edm is defined using code and there is a helper class to do that *ODataConventionModelBuilder*. 

Here's a snippet to define our two entites.

```c#
public static void Register(HttpConfiguration config)
{
    ...
    config.MapODataServiceRoute("OdataRoute", "odata", GetEdmModel());
}
...
private static IEdmModel GetEdmModel()
{
    var builder = new ODataConventionModelBuilder();
    builder.Namespace = "sample";
    builder.ContainerName = "SampleContainer";
    builder.EntitySet<Category>("Categories");
    builder.EntitySet<Product>("Products");
    return builder.GetEdmModel();
}
```

## Routing and conventions 

Odata service methods are defined on controller classes derived from **ODataController**.

For defining the routes, attribute based routing is used: **ODataRoutePrefix** attribute on the controller to define the route prefix of all actions, **ODataRoute** attribute on the controller action to define the action's route.

Mapping of http actions to the controller routes :

| Http Method | Action                   | Route example      |
| ----------- |--------------------------| -------------------|
| GET         | Get entity set           | /Categories        |
| GET         | Get entity by key        | /Categories({key}) |
| POST        | Create an entity         | /Categories        |
| PUT         | Update an entity         | /Categories({key}) |
| PATCH       | Patch an entity          | /Categories({key}) |
| DELETE      | Delete an entity         | /Categories({key}) |

## Basic crud operations

Rundown for implementing crud operations: 

- Add a new controller action. Call it anything as long as the route is acording to the convention.
- Define the route. The complte route is the concatenation of the values set in the ODataRoutePrefix and the ODataRoute
- Restrict the allowed http method, works without it too but makes for a clearer code.
- Execute the action against the database
- Return an action result, preferably using one of the existing helper methods.

### Implement get entity set

Sample code to list all entities of a kind: 

```c#
[ODataRoutePrefix("Categories")]
public class CategoriesController : ODataController
{
    ProductsDbContext dbContext = new ProductsDbContext();
    ...

    [ODataRoute()]
    [HttpGet]
    public IHttpActionResult Get()
    {
        return Ok(dbContext.Categories);
    }
```

Invoke from powershell:

```powershell
Invoke-WebRequest -URI http://localhost:61162/odata/Categories -Method Get 
```

### Implement get by key 

Sample code to lookup an entity by key:

```c#
[ODataRoute("({key})")]
[HttpGet]
public IHttpActionResult Get([FromODataUri] int key)
{
    var dbCategory = dbContext.Categories.SingleOrDefault(c => c.Id == key);

    if (dbCategory == null)
        return NotFound();

    return Ok(dbCategory);
}
```

Invoke from powershell:

```powershell
Invoke-WebRequest -URI 'http://localhost:61162/odata/Categories(1)' -Method Get 
```

### Implement create entity

Sample code to create an entity:

```c#
[ODataRoute()]
[HttpPost]
public IHttpActionResult AddCategory(Category category)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    dbContext.Categories.Add(category);
    dbContext.SaveChanges();

    return Created(category);
}
```

Invoke from powershell:

```powershell
$body = @{ Name ='NewCategory'} | ConvertTo-Json
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories" -Method Post -ContentType "application/json" -Body $body
```

### Implement update entity 

Sample code to update an entity:

```c#
[ODataRoute("({key})")]
[HttpPut]
public IHttpActionResult UpdateCategory([FromODataUri] int key, Category category)
{
    var dbCategory = dbContext.Categories
        .SingleOrDefault(c => c.Id == key);

    if (dbCategory == null)
        return NotFound();

    category.Id = key;
    dbContext.Entry(dbCategory).CurrentValues.SetValues(category);
    dbContext.SaveChanges();

    return StatusCode(System.Net.HttpStatusCode.NoContent);
}
```

Remark: The id supplied in the query is leading, that's why it's overriting the id that might be supplied in the request body. 

Invoke from powershell:

```powershell
$body = @{ Name ='NewCategory'} | ConvertTo-Json
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories(1)" -Method Put -ContentType "application/json" -Body $body
```

### Implement patch update entity 


Sample code to partially update an entity (patch):

```c#
[ODataRoute("({key})")]
[HttpPatch]
public IHttpActionResult UpdateDeltaCategory([FromODataUri] int key, Delta<Category> patch)
{
    var dbCategory = dbContext.Categories
        .SingleOrDefault(c => c.Id == key);

    if (dbCategory == null)
        return NotFound();

    patch.Patch(dbCategory);
    dbContext.SaveChanges();

    return StatusCode(System.Net.HttpStatusCode.NoContent);
}
```

Remark: the difference between update and patch is that update replaces the complete entity while patch updates only some of properties.

Invoke from powershell:

```powershell
$body = @{ Name ='NewCategory'} | ConvertTo-Json
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories(1)" -Method Patch -ContentType "application/json" -Body $body
```

### Implement delete

Sample code to delete an entity:  

```c#
[ODataRoute("({key})")]
[HttpDelete]
public IHttpActionResult Delete([FromODataUri] int key)
{
    var dbCategory = dbContext.Categories.Include("Products")
        .SingleOrDefault(c => c.Id == key);

    if (dbCategory == null)
        return NotFound();

    //TODO: remove foreign key relationships
    dbContext.Categories.Remove(dbCategory);
    dbContext.SaveChanges();

    return StatusCode(System.Net.HttpStatusCode.NoContent);
}
```

Remark: Setting foreign keys to null is not implemented in this version.

Invoke from powershell:

```powershell
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories(1)" -Method Delete
```

## Navigation and entity references

Navigating from categories to products. Access products belonging to a category when the category key is known. Url example: GET /Category(1)/Products

Code sample:

```c#
[ODataRoute("({key})/Products")]
[HttpGet]
public IHttpActionResult GetCategoryProducts([FromODataUri] int key)
{
    var dbCategory = dbContext.Categories.Include("Products")
        .SingleOrDefault(c => c.Id == key);

    if (dbCategory == null)
        return NotFound();

    return Ok(dbCategory.Products);
}
```

Invoke from powershell:

```powershell
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories(1)/Products" -Method Get 
```

And the reverse from products to category. Access the category of a product when the product key is known. Url example: /Products(1)/Category  

Code sample: 

```c#
[ODataRoute("({key})/Category")]
[HttpGet]
public IHttpActionResult GetCategory([FromODataUri] int key)
{
    var dbProduct = dbContext.Products.Include("Category")
        .SingleOrDefault(c => c.Id == key);

    if (dbProduct == null)
        return NotFound();

    return Ok(dbProduct.Category);
}
```

Invoke from powershell:

```powershell
Invoke-WebRequest -URI "http://localhost:61162/odata/Products(1)/Category" -Method Get 
```

## Queries 

The following query capabilities are supported by odata: 

| keyword      | function                 
| ------------ |-----------------------------------------------------------------------------------|
| $expand      | Expands related entities inline.                                                  |
| $filter      | Filters the results, based on a Boolean condition.                                |
| $count       | Tells the server to include the total count of matching entities in the response. |
| $orderby     | Sorts the results.                                                                |
| $select      | Selects which properties to include in the response.                              |
| $skip        | Skips the first n results.                                                        |
| $top         | Returns only the first n the results.                                             |


Fist the query capabilities have to be enabled in the edm. Could not find a global switch for that so I will do it individually on an entity level. Here's how to enable various query capabilities: 

```c#
private static IEdmModel GetEdmModel()
{
    ...
    builder.EntitySet<Category>("Categories").EntityType
        .HasKey(e => e.Id).Select().Filter().Expand().OrderBy().Page().Count(); 
    ...
}
```

Then the get all actions have to adjusted to support queries. Add the *EnableQuery* attribute for that.


```c#
[EnableQuery()]
[ODataRoute()]
[HttpGet]
public IHttpActionResult Get(ODataQueryOptions<Category> options)
{
    return Ok(dbContext.Categories);
}
```

Remark: the query options will be passed down to to the sql server

Invoke from powershell:

```powershell
Invoke-WebRequest -URI "http://localhost:61162/odata/Categories?$filter=Name eq 'Goods'&$select=Name&$top=1&$skip=1&$expand=Products" -Method Get 
```

## Testing

For testing the endpoints I use powershell or [postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en).

## References

- Github source code [link](https://github.com/adam-gligor/archeology/tree/master/OdData4Sample-master/OdData4Sample-master)
- The Odata standard [link](http://docs.oasis-open.org)
- Documentation on using on odata4 with web api [link](http://odata.github.io/WebApi)
