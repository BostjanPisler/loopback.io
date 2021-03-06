---
title: "Xamarin client API"
lang: en
layout: page
keywords: LoopBack
tags:
sidebar: lb3_sidebar
permalink: /doc/en/lb3/Xamarin-client-API.html
summary:
---

## Setup

You must have a C# development environment, such as Visual Studio or Xamarin Studio.
Add to your project the LBXamarinSDK.cs file generated by the lb-xm command.

You must add the following NuGet packages to your project:

* [RestSharp Portable](https://www.nuget.org/packages/FubarCoder.RestSharp.Portable/3.0.0-alpha4)
* [Newtonsoft JSON](https://www.nuget.org/packages/Newtonsoft.Json/)

You now have the Xamarin SDK customized for your LoopBack server available in your project.

## Overview

The SDK has three entities:

* **Models** - Local instances of the LoopBack models with the same name as the Loopback Models; for example, "Car."  
  They are in the `LBXamarinSDK` namespace.

* **Repositories** - Classes unique to each type of model. They contain the methods that exist on the server: create, retrieve, update,
  and delete methods and the custom methods that were added to the Loopback models. Client code uses Repositories to perform server calls. 
  They have the plural name of the Loopback model; for example, "Cars."  They are in the `LBXamarinSDK.LBRepo` namespace.

* **Gateway** - A class containing the code that enables the repositories to talk to the server. It is in the `LBXamarinSDK` namespace.

## Models

Models in C# have the same name as the Loopback models.

### Creating a local instance

The model is a C# class. Use it and create it as you would any other class. 

```
Car myCar = new Car()
{
    wheels = 5,
    drivers = 3,
    name = "blarg"
}
```

{% include important.html content="

Again, this creation is only local. To perform server calls use the repositories.

" %}

## Repositories

Repositories have the plural name of the Loopback models. They contain the methods that exist on the Loopback server.

For example, if **Car** is the name of the model, then **Cars** is the name of the repository:

```
// Creation of a local instance
Car myCar = new Car();
myCar.accidents = 0;

// Repository method. Persisting local instance to Loopback server
Cars.Create(myCar);
```

The following sections introduce the methods in the repositories.

### Create, retrieve, update, and delete methods

The repositories have the same basic model create, read, update, and delete methods for all models, as is in the Loopback server REST API.

{% include important.html content="

The method createChangeStream() is still unsupported.

" %}

#### Count
```
Task<int> Count(string whereFilter = "")
```

Counts instances of the model that match the specified where filter. 

Parameter: 

* Optional filter, counting all instances that this filter passes. 

Returns the number of matching instances.  

For example: 

```
int numOfCustomers = await Customers.Count();
int numOfSmartCustomers = await Customers.Count("{\"where\":{\"and\": [{\"pizzas\":\"500\"}, {\"cats\":\"500\"}]}}");
```

#### Create 

```
Task<T> Create(T _theModel_)
```

Creates a new instance of the model and persist it into the data source. T is a generic template (the same for all loopback models in the SDK).

Parameter:

* `_theModel_`: The model to create.

Returns the created model.

For example:

```
c = await Customers.Create(c);
```

#### DeleteById

```
Task DeleteById(String Id)
```

Deletes a model instance by ID from the data source.

Parameter:

* _`Id`_ - String ID of the model to delete.

For example:

```
await Customers.DeleteById("1255");
```

#### Exists

```
Task<bool> Exists(string _Id_)
```

Checks whether a model instance exists in the data source.

Parameter:

* `_Id_` - the String ID of the model of interest.

Returns a Boolean value.

For example:

```
if(await Customers.Exists("441")) { ... }
```

#### Find 

```
Task<IList<T>> Find(string filter = "") 
```

Finds all instances of the model matched by filter from the data source. A filter is applicable here. 

Parameter: 

* an optional filter to apply on the search. 

Returns a list of all found instances. 

For example:  

```
IList<Customer> l1 = await Customers.Find();
IList<Customer> l2 = await Customers.Find("{\"where\":{\"and\": [{\"pizzas\":\"500\"}, {\"cats\":\"500\"}]}}");
```

#### FindById

```
Task<T> FindById(String _Id_) 
```

Finds a model instance by ID from the data source.

Parameter:

* `_Id_` - String ID of the model to find.

Returns the found model.

For example:

```
Customer c = await Customers.FindById("19");
```

#### FindOne

```
Task<T> FindOne(string filter = "")
```

Finds first instance of the model matched by filter from the data source.

Parameter:

* Optional filter to apply on the search.

Returns the found model

For example:

```
Customer c = await Customers.FindOne();
Customer c = await Customers.FindOne("{\"where\":{\"and\": [{\"pizzas\":\"500\"}, {\"cats\":\"500\"}]}}");
```

#### UpdateAll

```
Task UpdateAll(T updateModel, string _whereFilter_)
```

Updates instances of the model matched by where from the data source.

Parameters:

* `_updateModel_` - Update pattern to apply to all models that pass the where filter
* `_whereFilter_` - Where filter. 

For example: 

```
Customer updatePattern = new Customer() {cars = 99};
await UpdateAll(updatePattern, "{\"where\":{\"and\": [{\"pizzas\":\"500\"}, {\"cats\":\"500\"}]}}");
```

this updates all customers who have 500 cats and 500 pizzas to have 99 cars.

#### UpdateById

```
Task<T> UpdateById(String _Id_, T _update_) 
```
Updates attributes of a model instance and persists it to the data source.

Parameters:

* `_Id_` - String ID of the model to update
* `_update_` - the update to apply.

Returns the updated model.

For example:

```
Customer c = new Customer() { hp = 100 };
h = await UpdateById("11", c);
```

There are more methods created automatically if relations between models are added. Their definition and description are in the generated code.

#### Upsert 

```
Task<T> Upsert(T _theModel_) 
```

Updates an existing model instance or insert a new one into the data source. 

Parameter: 

* `_theModel_`, the model to update/insert.

Returns the updated/inserted model. 

For example: 

```
c = await Customers.Upsert(c);
```

### Relation Methods

The SDK will create C# code for methods that are created for models connected by relations, just like in the REST API.

#### Example

Suppose we have two Loopback models: Goal and TodoTask, and the relation between them is that TodoTask [belongs to](BelongsTo-relations.html) Goal.

Then the SDK will create, among others, the method to fetch the goal of a given todo task. You can see the generated code in the TodoTasks repository:

```
/*
 * Fetches belongsTo relation goal
 */
public static async Task<Goal> getGoal(string id, [optional parameters])
{
...
}
```

Then a usage example is:

```
var TodoTaskID = "10034";
Goal relatedGoal = await TodoTasks.getGoal(TodoTaskID);
```

### Custom remote methods

The LoopBack Xamarin SDK supports custom remote methods and will create C# code for them. These methods exist in the repository classes.

#### Example

Suppose there is a LoopBack model called Logic. Then, if there is a custom method in the LoopBack application in **`Logic.js`** as follows:

```javascript
module.exports = function(Logic) {
  Logic.determineMeaning = function(str, cb) {
    console.log(str);
    cb(null, 42);
  };

  Logic.remoteMethod('determineMeaning', {
    accepts: {
      arg: 'str',
      type: 'string',
      http: {
        source: 'body'
      },
      required: true
    },
    returns: {
      arg: 'res',
      type: 'number',
      root: true
    },
    http: {
      path: '/determineMeaning',
      verb: 'post'
    },
    description: 'This is the description of the method'
  });
};
```

Then once we compile an SDK using lb-xm, a method will be created in the C# code of **LBXamarinSDK.cs**, with the signature:

```
Task<double> determineMeaning(string str)
```

If you go into the _Logic_s repository code you can see the generated code for the method:

```
/*
 * This is the description of the method
 */
public static async Task<double> determineMeaning(string str)
{
...
}
```

Then an example of using it: 

```
double something = await Logics.determineMeaning("blarg");
```

## Gateway

Gateway has several important methods.

### isConnected

```
Task<bool> isConnected(int _timeoutMilliseconds_ = 6000)
```

Checks if the server is connected.

Parameter:

* _timeoutMilliseconds_ - timeout for the check, in ms.

Returns a Boolean value.

```
connectedStatus = await Gateway.isConnected();
```

There are additional Gateway methods with a descriptions in the generated code. 

### SetAccessToken 

```
void SetAccessToken(AccessToken _accessToken_) 
```
All server calls will be done with the authorization given by the supplied access token.

```
var list1 = await Users.Find(); // error 401, not authorized
AccessToken token = await Users.login(auth); // Server call: Performing login
Gateway.SetAccessToken(token);
var list2 = await Users.Find(); // ok, list is returned
```

### SetDebugMode

This method sends output when important SDK functionality is executed (output from `System.Diagnostics.Debug`) to the console.

```
void SetDebugMode(bool _isDebugMode_)
```

Once debug mode is set to true, important information will be sent to the debug console.

For example, if we make a server call:

```
Customer myCustomer = new Customer() 
{
   name = "joe",
   geoLocation = new GeoPoint() 
   {
      Longitude = 0,
      Latitude = 11
   }
};
myCustomer = await Customers.Create(myCustomer);
```

Then when the debug mode is on, calling the above code will display the following output to the console:

```
-------- >> DEBUG: Performing POST request at URL:
'http://10.0.0.1:3000/api/Customers', Json: {"geoLocation":{"lat":11.0,"lng":0.0},"name":"joe"}
```

### SetServerBaseURL

```
void SetServerBaseURL(Uri _baseUrl_)
```

Parameter:

* baseUrl - URL of the server API. 

Sets the API URL of the loopback server. For example:

```
SetServerBaseURL(new Uri("http://10.0.0.1:3000/api/"));
```

### SetTimeout

```
void SetTimeout(int _timeoutMilliseconds_ = 6000)
```

Sets the timeout of server calls. if no response is received from server after this period of time, the call will die.

```
Gateway.SetTimeout(2000);
```

## Handling exceptions

Perform all access to a repository (calls to methods that communicate with the server) inside a `try / catch` clause, including a clause to catch a `RestException`. 

`RestException` has a StatusCode field that specifies the return status of the error.

For example: 

```
User myUser = new User()
{
  email = "j@g.com",
  password = "123"
};
try 
{
  myUser = await Users.Create(myUser); // user created on the server. success.
}
catch(RestException e) 
{
	if(e.StatusCode == 401)
	{
    // not authorised to create the user
	} 
	else if(e.StatusCode == 422) 
	{
    // unprocessable entity. Perhaps the email already exists, perhaps no password 
    // is specified inside the model myUser, etc
	}
}
```
