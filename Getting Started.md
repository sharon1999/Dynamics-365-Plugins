# Why use Plugins?
![image](https://user-images.githubusercontent.com/49652785/172536700-06f8510f-017c-43ff-a335-75ade4c73d4a.png)

## Could we use a javascript? Or a workflow? Or a business rule?
* Javascripts will run on the client side only. So, even if we had validations there, they would not work for 
the custom applications.
* The business rules and workflows – you can use those for server-side validations, too. However, business rules can be configured and cannot be customized through 
development. There is only so much you can configure there, and you certainly can’t do complex calculations or queries.
* Workflows can be configured and customized (using custom workflow activities).

### Plugins are still somewhat more flexible when it comes to configuring the trigger conditions, pre-images, post-images, messages.
***
#### Still, when developing a plugin you need to know the exectuion context – what was happening on the client, what is the record being processed, what are the fields being updated, what is the action being taken.

## Sample Code 
 * Code to check if name is Test then throw an exception
 ```cs
 using System;
using Microsoft.Xrm.Sdk;
using Microsoft.Xrm.Sdk.Query;
namespace Training.Plugins
{
    public class ValidationPlugin : IPlugin
    {
        public void Execute(IServiceProvider serviceProvider)
        {
            IPluginExecutionContext context = (IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));
            Entity entity = (Entity)context.InputParameters["Target"];
            string name = (string)entity["ita_name"];
            if ("Test".Equals(name))
            {
                throw new InvalidPluginExecutionException("Cannot use this name");
            }
        }
    }
}
```

### Every plugin is expected to implement “Execute” method of the IPlugin interface, and there is a single parameter passed to that call: 
```cs
  public void Execute(IServiceProvider serviceProvider)
  {
  }
```        
* ### IServiceProvide is a common interface not specific to Dynamics that defined just one method

* ### Using the serviceProvide, you can get access to the contextual information specific to that plugin execution:
```cs
  IPluginExecutionContext context = (IPluginExecutionContext) serviceProvider.GetService(typeof(IPluginExecutionContext));
  Entity entity = (Entity)context.InputParameters["Target"];
```
* First, we are getting the context in exactly the way we just discussed. And, then, we are getting the “Target” from the context.InputParameters
* “Target” is, really, the in-memory representation of the data that’s being updated/created/deleted/etc. 
* In other words, if, in the user interface, you are updating the account record, then, in the plugin, “Target” will represent exactly that account. 
### IPluginExecutionContext gives you access to pretty much all the contextual details you may get access to in the plugin:
- context.InputParameters[“Target”]
- context.PreEntityImages
- context.PostEntityImages
- context.MessageName
- context.Stage
 
>Take a look at the MSDN page describing IPluginExecutionContext in more details  
https://msdn.microsoft.com/en-us/library/microsoft.xrm.sdk.ipluginexecutioncontext.aspx
 ***
> ### Q1: Will the context be there for every plugin execution?
* Yes, it will.
> ### Q2: Will the Target be there for every plugin execution?
* No, not necessarily. It depends on the plugin step configuration. Remember we can register a plugin for different messages? Not all of them will have a target. Although, messages like “Create”/”Update”/”Delete” will have a Target. 
> ### Q2.1: How do you know what you will find in the “Target”? 
* You just learn those things as you keep developing plugins. For example, for the “Create” and “Update” plugins, you will always have a “Target”, and it will be of “Entity” type. For the “Delete” plugins, you will also have a “Target”, but it will be of “EntityReference” type.
 ***
# Use of Contain vs Get Attribute Method
- You can use Contains method:
```cs
if (entity.Contains("ita_name"))
 {
 string name = (string)entity["ita_name"];
 if ("Test".Equals(name))
 {
 throw new InvalidPluginExecutionException("Cannot use this name");
 }
 }
 ```
- You can re-write the code using GetAttributeValue method:
```cs
 string name = entity.GetAttributeValue<string>("ita_name");
 if ("Test".Equals(name))
 {
 throw new InvalidPluginExecutionException("Cannot use this name");
 }
```
>The difference between those two is that GetAttributeValue will not tell you if Name field was 
populated or not. It will give you a value or null. However, null can also be a valid value when the field is 
being cleared. You might not be able to clear the “Name” field since it’s a required field, but, if we were 
using an optional field, you might just make it empty on the update. In the plugin, you would have that 
attribute in the Target, though the value you’d find null value there.
* So, for our current scenario either of those methods above would work. But, in general, whenever you 
need to check if a field has been modified as part of the Update/provided as part of the Create, you 
should be using “Contains”.

### Re-write the plugin using Contains
```cs
using System;
using Microsoft.Xrm.Sdk;
using Microsoft.Xrm.Sdk.Query;
namespace Training.Plugins
{
   public class ValidationPlugin : IPlugin
   {
     public void Execute(IServiceProvider serviceProvider)
     {
       IPluginExecutionContext context = (IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));
       Entity entity = (Entity)context.InputParameters["Target"];
       if (entity.Contains("ita_name"))
       {
         string name = (string)entity["ita_name"];
         if ("Test".Equals(name))
         {
            throw new InvalidPluginExecutionException("Cannot use this name");
         }
       }
     }
   }
}
```
***
# OrganizationService
> ### Organization Service represents a connection to Dynamics. It helps when we have access to the Target, and it’s certainly useful when we can do the validations, but that’s not what plugins are all about. What really makes them work for lots of different scenarios is that we get access to the so-called Organization Service.
> ### With the Organization Service, you can, basically, execute requests. You can query data, you can delete data, you can create, assign, update, associate data. You can query the metadata, you can publish metadata changes, you can even create new entities, fields, option set values, etc.

### In general, Organization Service uses Execute method to execute such requests, and that method returns a response object as a result.
```cs
  // Obtain the organization service reference which you will need for web service calls.
  IOrganizationServiceFactory serviceFactory = (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));
  IOrganizationService service = serviceFactory.CreateOrganizationService(context.UserId);
```
# The entire code will be 
```cs
  public void Execute(IServiceProvider serviceProvider)
  {
     IPluginExecutionContext context =  (IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));
     IOrganizationServiceFactory serviceFactory = (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));
     IOrganizationService service = serviceFactory.CreateOrganizationService(context.UserId);
  }
 ```
 # Once we have an instance of IOrganizationService, there is a bunch of useful methods available to us:
https://msdn.microsoft.com/en-us/library/microsoft.xrm.sdk.iorganizationservice.aspx

| Name        | Description     | 
| ------------- |:-------------:| 
| Associate(String, Guid, Relationship, EntityReferenceCollection)	      | Creates a link between records. | 
| 	Create(Entity)  | Creates a record.     |   
| Delete(String, Guid) | Deletes a record.     |    
|Disassociate(String, Guid, Relationship, EntityReferenceCollection) |Deletes a link between records.     |  
| Execute(OrganizationRequest) | Executes a message in the form of a request, and returns a response.     |  
| Retrieve(String, Guid, ColumnSet) | Retrieves a record.     |  
| RetrieveMultiple(QueryBase) | Retrieves a collection of records.    |  
| Update(Entity) | Updates an existing record.   |  



	
