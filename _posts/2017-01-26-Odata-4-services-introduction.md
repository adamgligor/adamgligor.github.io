---
layout: post
title:  Odata 4 service tutorial - part 1
date: '2017-01-29'
tags: odata
---


Geting started with OData 4 services. Here's a quick guide to geting started ... 

## What is odata

OData (Open Data Protocol) is an OASIS standard that defines a set of best practices for building and consuming RESTful APIs. 
OData helps you focus on your business logic while building RESTful APIs without having to worry about the various approaches to define request and response headers, status codes, HTTP methods, URL conventions, media types, payload formats, query options, etc. 
OData also provides guidance for tracking changes, defining functions/actions for reusable procedures, and sending asynchronous/batch requests.

Source: [odata.org](http://www.odata.org/)

## Prerequisites

For the purpuse of ilustrating basic capabilities of an odata service and some of the the implementation details, I created a demo project. the complet source code can be foind on github, see references section.

The data model is very simple and consists of two entities **categories**, and **products**, with one to many relationship between them.

Category: Id, Name, Products

Product: Id, Name, Price, Cateogory

The example code will cover the following topics: 

 - Basic crud operations
 - Navigation and entity references
 - Queries 

The code was built with: visual studio 2015, .net 4.6.2, and odata 4, web api 2.2, entity framework 6 and sql local db.


## Initial setup and gotchas

Get the sample code  
Install odata nuget packages, .. gotchas
Install and configure entity framework 6  beyond scope


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

Http request example: 

- headers 
- url 
- body 

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

### Implement update entity 

Sample code to update an entity:

```c#
    [ODataRoute("({key})")]
    [HttpPut]
    public IHttpActionResult UpdateCategory([FromODataUri] int key, Category category)
    {
        var dbCategory = dbContext.Categories.SingleOrDefault(c => c.Id == key);

        if (dbCategory == null)
            return NotFound();

        category.Id = key;
        dbContext.Entry(dbCategory).CurrentValues.SetValues(category);
        dbContext.SaveChanges();

        return StatusCode(System.Net.HttpStatusCode.NoContent);
    }
```

Remark: The id supplied in the query is leading, that's why it's overriting the id that might be supplied in the request body. 


### Implement patch update entity 


Sample code to partially update an entity (patch):

```c#
    [ODataRoute("({key})")]
    [HttpPatch]
    public IHttpActionResult UpdateDeltaCategory([FromODataUri] int key, Delta<Category> patch)
    {
        var dbCategory = dbContext.Categories.SingleOrDefault(c => c.Id == key);

        if (dbCategory == null)
            return NotFound();

        patch.Patch(dbCategory);
        dbContext.SaveChanges();

        return StatusCode(System.Net.HttpStatusCode.NoContent);
    }
```

Remark: the difference between update and patch is that update replaces the complete entity while patch updates only some of properties.

### Implement delete

Sample code to delete an entity:  

```c#
    [ODataRoute("({key})")]
    [HttpDelete]
    public IHttpActionResult Delete([FromODataUri] int key)
    {
        var dbCategory = dbContext.Categories.Include("Products").SingleOrDefault(c => c.Id == key);

        if (dbCategory == null)
            return NotFound();

        //TODO: remove foreign key relationships
        dbContext.Categories.Remove(dbCategory);
        dbContext.SaveChanges();

        return StatusCode(System.Net.HttpStatusCode.NoContent);
    }
```

Remark: Setting foreign keys to null is not implemented in this version.

## Navigation and entity references

Navigating from categories to products. Access products belonging to a category when the category key is known. Url example: GET /Category(1)/Products

Code sample:

```c#
    [ODataRoute("({key})/Products")]
    [HttpGet]
    public IHttpActionResult GetCategoryProducts([FromODataUri] int key)
    {
        var dbCategory = dbContext.Categories.Include("Products").SingleOrDefault(c => c.Id == key);

        if (dbCategory == null)
            return NotFound();

        return Ok(dbCategory.Products);
    }
```

And the reverse from products to category. Access the category of a product when the product key is known. Url example: /Products(1)/Category  

Code sample: 

```c#
    [ODataRoute("({key})/Category")]
    [HttpGet]
    public IHttpActionResult GetCategory([FromODataUri] int key)
    {
        var dbProduct = dbContext.Products.Include("Category").SingleOrDefault(c => c.Id == key);

        if (dbProduct == null)
            return NotFound();

        return Ok(dbProduct.Category);
    }
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


Fist the query capabilities have to be enabled in the edm. 
TODO: how to globally enable them

```c#
    private static IEdmModel GetEdmModel()
    {
       ...
        builder.EntitySet<Category>("Categories").EntityType
            .HasKey(e => e.Id).Select().Filter().Expand().OrderBy().Count(); 
       ...
    }
```

Then the get actions have to adjusted to support queries.

 - Add the *EnableQuery* attribute
 - Pass the query down to the database

```c#
    [EnableQuery()]
    [ODataRoute()]
    [HttpGet]
    public IHttpActionResult Get(ODataQueryOptions<Category> options)
    {
        ODataQuerySettings settings = new ODataQuerySettings() { PageSize = 10 };
        IQueryable dbCategories = options.ApplyTo(dbContext.Categories, settings);
        return Ok(dbCategories);
    }
```


## Testing

Postman , sample test http request provided.

## References

Github examples 
Postman collection to test the urls 
Source: [wikipedia](https://en.wikipedia.org/wiki/Inversion_of_control)


http://docs.oasis-open.org/odata/odata/v4.0/odata-v4.0-part2-url-conventions.html

