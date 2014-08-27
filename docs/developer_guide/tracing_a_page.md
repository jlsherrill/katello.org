#Tracing A Page

Katello features two different types of pages, ones that are javascript based and ones that that are rails based.

Javascript based pages use AngularJS and communicate with the Satellite's rest api.  This is the same api that hammer uses, and the same api that a user would script against for automation.

Rails based pages (primary Foreman) are handled completely by the rails server.

The Rest API is very similar to the normal Rails based pages, the only difference is the return of JSON data instead of rendered HTML.

## Simple rails Primer

Rails is a Model View Controller (MVC) framework.   What does that mean?

There are 3 basic entities (although I'll add a 4th):

Model - The buisiness logic of the application.  A User is a model (also a 'class' in programming terms).  You create a user and it gets saved to the database.
  Models live in the /app/models/  directory of an application.

View - What to actually display to the user.  These are defined in the /app/views directory of the application.

Controller - The logic that takes input from the user (in whatever format), and does one or more of the following:
  * Looks up the correct model(s)
  * Performs some action on one or more models
  * Instructs the server what view to render
  Controllers have one or more actions such as 'create', 'show', 'update', 'destroy' (common CRUD actions) and other custom defined actions.  Actions are simply methods within the controller class.  Controllers live in /app/controllers within the application.
  
Route - An entry that maps a HTTP method (get, post, put, delete) and a URL (such as /foo/bar), into a controller and an action.  It may also be responsibile for transforming parts of the url parameters.  Such as 'GET /hosts/3' would match the 'GET /hosts/:id' route and call hosts_controller#show providing a parameter of 'id' with a value of 3.


Note for katello and foreman models, views, and controllers may exist in /usr/share/foreman/app/ or /opt/rh/ruby193/root/usr/share/gems/gems/katello-VERSION/app.  You may need to check both places to find the file you are looking for.


## Determining if a page is Javascript or Rails

There isn't a simple way of doing this, however the easiest is to view the page's source in the browser (right click, "View Page Source") and look for the word "angular".  For example, the following will appear at the top of every AngularJS Based page:

```
<script type='text/javascript'>
angular.module('Bastion').value('currentLocale', 'en');
angular.module('Bastion').value('CurrentOrganization', "1");
angular.module('Bastion').value('CurrentUser', "1");


</script>

```

A second method would simply to open the javascript console and run 'angular'.  If 'undefined' or "angular is not defined" is printed, that particular page is NOT a javascript based page.


## Tracing a rails page

Lets take an example page such as the new Hosts page.

The URL:  "/hosts/new" 

From the url i can guess that this is probably in the hosts_controller, on a 'new' action.  Let check:


```
> foreman-rake routes | grep host

<SNIP>
new_host GET    /hosts/new(.:format)    hosts#new {:id=>/[^\/]+/}
<SNIP>
```

Running 'foreman-rake routes' on production server will print all of the urls and what controller they go to.

What the above is telling us is that this is going to the hosts_controller and using the 'new' action.  "hosts#new" implies this.  Keep in mind that "GET /hosts/3" would go to a different action that "PUT /hosts/3".  The http method is very important.


### More complex routing example:

Lets look at a more tricky example:

```
host_facts GET    /hosts/:host_id/facts(.:format)  fact_values#index {:id=>/[^\/]+/, :host_id=>/[^\/]+/}
```

While you might have expected this to go to the hosts_controller but it does not.  It actually goes to the fact_values controller to the index method.


### Rendering the view

For an action such as 'show' a corresponding view is likely to be rendered.  For example, hosts_controller#show would render /usr/share/foreman/app/views/hosts/show.html.erb by default. 

Sometimes this is done automatically, such is the case for hosts_controller#new.  You'll notice in /usr/share/foreman/app/controllers/hosts_controller.rb there is no 'new' method. Yes everything still works as if there was.  Some built in 'actions' don't actually require you to define a controller action if you don't need to do anything other than render a view of the same name. 

Some controller actions may render a different template such as a partial or even pure json data.  If you see:

```
render :json => {:foo => 'bar'}
```

This is rendering pure json and does not need a view.

If you see something like:

```
render :partial => :foo
```
This will render a partial view.  A partial is a view that does not need additional things like a navigation structure.  It is meant for use on pages where you need to fetch part of the page from the server an inject it into a page that is already rendered.  If the above render statement was in an action in the hosts controller you would fine that partial in '/app/hosts/_foo.erb.html'. (Note partial filenames start with an underscore).



## Tracing an API call

A call to the API is very similar to a normal UI call with one major exception: It is not rendering HTML but JSON, and when it does render JSON it uses a special type of view called 'rabl'.






## Tracing a Javascript Page
