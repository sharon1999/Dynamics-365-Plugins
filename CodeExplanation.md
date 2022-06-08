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
 
