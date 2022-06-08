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

