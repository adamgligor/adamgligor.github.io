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

I will be creating a service for the purpuse of ilustrating basic capabilities and the implementation details.
Topic covered: 

 - Basic crud operations
 - Navigation and entity references
 - Queries 

The sample code was built in visual studio 2015 with .net 4.6.2, and odata 4, web api 2.2, entity framework 6 and sql local db for presistance.

Get the sample code  
Install odata nuget packages, .. gotchas
Install and configure entity framework 6  beyond scope


## The data model

Starting off with the model, two entities: **categories**, and **products** with one to many relationship between them.

Category: Id, Name, Products

Product: Id, Name, Price, Cateogory


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
| PATCH       | Update (delta) an entity | /Categories({key}) |
| DELETE      | Delete an entity         | /Categories({key}) |

## Basic crud operations

Rundown for implementing crud operations: 

- Add a new controller action. Call it anything as long as the route is acording to the convention.
- Define the route. The complte route is the concatenation of the values set in the ODataRoutePrefix and the ODataRoute
- Restrict the allowed http method, works without it too but makes for a clearer code.
- Execute the action against the database
- Return an action result, preferably using one of the existing helper methods.

### Implement get entity set

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

### Implement get all 

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


### Implement delta update entity 

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

## Navigation and entity references

## Queries 


## Testing


## References

Source: [wikipedia](https://en.wikipedia.org/wiki/Inversion_of_control)

