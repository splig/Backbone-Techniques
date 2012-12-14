# Introduction

# General Techniques
## Return Promises for Fetch and Save
### Problem:
Eliminate repetitive success and error callbacks.  
### Solution: 
Backbone provides several hooks where you can tweak a model's CRUD implementation.  Backbone provides 2 methods for populating models: fetch and save.  These objects by default return the xhr object which is marginally useful.  You can change these methods to return something more useful, such as a promise.
```javascript
// Set the default implementation of `Backbone.ajax` to proxy through to $
Backbone.ajax = function (options) {
    //the promise should be set in Backbone.sync, this is a Q promise
    var deferred = options.deferred;

    Backbone.$.ajax.apply(Backbone.$, arguments);

    return deferred.promise;
};
```
## Implementing a Controller Object
### Problem:
You have a large application and want to avoid placing logic in routes and views.
### Solution: 
A possible solution for this is to introduce a supervising controller object.  The controller can be a simple object that exposes methods or actions that will be invoked based on the current route.
```javascript
var controller = {
  index : function(){
    var view = new indexView();
    view.render($('#content'));
  },
  detail : function(albumId){
    var model = new AlbumModel({albumId:albumId});
    model.fetch({success:function(response, status, xhr){
      var detailView = new DetailView({model : model});
      detailView.render($('#content'));
    }})
    
  }
};
```

### Dynamic Controller Invocation
#### Problem
Your routes are basically repeating the same logic - creating a controller object, then calling the action method, you want something a little more DRYful.
#### Solution 
If you are using a controller like object, it is farily trivial to implement a standard routing system that works like some of the server side frameworks such as ASP.NET MVC.  In the following example, we implement a dynamic route based on the url where the first part of the path defines the controller and the second part defines the action.  

```javascript
crossroads.addRoute('/{controller}/{action}/{args}', function(controller, action, args){
  require([controller], function(ControllerObject){
    var myController = new ControllerObject();
    myController[action] && myController[action].call(myController, args);
  });
});
```

Note, the above implementation uses crossroads as opposed to the standard backbone router. Mixing and matching components like this is not discouraged.  Backbone's flexibility allows you to architext a solution which best fits your needs. 

Also note that we are using requirejs to pull in the controller module, we then invoke the controller's action method using the call method, this allows us to define the controller as the 'this' variable.

## Dependency Injection
I highly reccomend you consider using a module loader such as RequireJS.
## Using Mixins
### Problem:
You want to share functionality between objects.
### Solution:
One possible way to share objects is to add properties to the target object's prototype.  A drawback of this approach is that it is somewhat clunky and complicated, and also forces implementors to use new for object creation.

A lighter and more elegant solution is to use the concept of a mixin, which essentially just copies target properties from one object to another.  This method favors composition over inheritance, which is preferable in most code reuse situations.

```javascript
var loggingFunctions = {
  log = function(message){
    console.log(message);
  }
},
LoggingView = Backbone.View.extend($.extend({}, loggingFunctions),{
  render = function(){
    
  }
});

myView = new LoggingView();
myView.log('test');
```
The example above illustrates a solution which could reuse the logging functions objects via composition.  The properties are copied to the target object by using jquery's extend method, underscore and other libraries have an equivellent method, which could be used as well.  The only caveat to this approach is that copies of the source object are applied to the target object, so if you are considering using a mixin to extend functionality to thousands of objects - it might make more sense to use the prototypal inheritance/new discussed earlier to save resources.

## Decorating instance methods/functions
### Problem:
You want to create a view that logs its output to the console whenever it's rendered, but you want it to automatically happen whenever render is called on a base view.  You don't want the developer to have to invoke the method excplicity in render.
### Solution:
Place render function in a temporary function, then call temporary function from within base render call.  Remember that if you require the views html to be present 
```javascript
//decorate render call so we can add the validation binding
view.oldRender = view.render; //store instance render (user provided render) in a property
view.render = function () { //now implement the base render method 
view.oldRender.apply(view, arguments);
//provide your decoration logic here
console.log('content is ' + this.$el.html());
});
```
# Model Techniques
## View Models
### Problem:
Most applications are basically just wrappers around what many refer to as a domain model.  A domain model can be thought of as business rules and data.  Usually the domain model does not have a 1-1 relationship with the UI or views that are presented to the user.  This presents a problem. How can we take advantage of the Model/View when the view looks nothing like the model?
### Solution:
A possible solution is to use a view model.  A view model is basically just a value object that represents properties used within the view.

## Using Backbone with Non-Restful services
### Problem:
You want to use backbone, but don't have RESTful web services.

Backbone works great with RESTful services, that is to say it works great as long as your services only needs a single url and HTTP verbs (GET,POST,DELTE,PUT) to differentiate the actions.  Many applications may not have the luxury of 100% RESTful services, but might still want to use backbone.

### Solution: 
Backbone gives you an all purpose hook called backbone.sync that you can use to override its standard - RESTful behaviour.  


## Modelbinding
Modelbinding is a concept where your model's attributes are automatically bound to elements on the view.  Modelbinding offers several benefits:
* eliminates need for tedious event wiring logic to keep view and model in sync.
* reduces total amount of code in application.
* encapsulates event wiring and model to view sync logic

### Model to view model binding
This is typically taken care of via a template library such as handlebars or mustache.

### View to model model binding
* Typically consists of binding a change events to form elements on page.  
* There are backbone extensions which can take care of this for you. 
* fairly easy to write your own. 

# View Techniques
## Dynamic Template Loading
## Validation
* Backbone's validation occurs on the model object.  
* Validate is provided by the developer, is called automatically before save, this gives the developer a hook to implement custom reusable logic.  There are some 3rd party extensions for this. For example:  Backbone.Validation 

## Base View
A base view can be useful in situations where you find yourself copy pasting the same code in each view or calling the same methods for each view; validation or template loading for example.




