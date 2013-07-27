{{$ meta }}
title: using angularjs with a back-end API
created: 2013-05-30
public: yes
author: <a href="http://twitter.com/laurihy">@laurihy</a>
disqusid: 7ae2236a-cf09-4c6e-a940-f482e6e0faff
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

In this walkthrough, we'll be storing the data for our lists app using [backlift.com](backlift.com)'s data API for persistence. We have already built a nifty little app for lists in the [previous tutorial](https://blog.backlift.com/entry/angular-tut1), and are now about to extend it. First, we'll wire up a backend to save and load lists and then modify the *router* so that it's possible to create new lists on the fly. 

Check out the final version: <https://angularfinal-ep2wx.backliftapp.com>

{{$ break }}

If you haven't read the previous part or aren't already familiar with AngularJS and Backlift, you might want to check it out. There we cover topics such as *controllers*, *views*, *routers*, *directives* and *two way data-bindings*. (Once again, you can find it [here](https://blog.backlift.com/entry/angular-tut1).)

### Why AngularJS and Backlift?

AngularJS is a powerful client-side framework for crafting modern web applications. Backlift takes care of the back-end and frees you to build great things on front-end. Using these two makes it really quick to get started, so they're ideal for both rapid prototyping and learning as well as building serious apps. 


## Setup

We'll continue pretty much straight from where we left in the previous part. This far we've built an app for listing items and you can either use the version you've already built, or create a new project from a Backlift template using this link:

<a target="top" href="https://www.backlift.com/backlift/dropbox/create?template=github.com/laurihy/AngularTutorial1-Final&appname=angulartemplate2"><button class="btn primary" style="font-size:larger;padding:5px"> Copy template to Dropbox </button></a>

This will add a new folder to your Dropbox with all required files. Also, the site is already live and you can view it by clicking "View app" once the app has been created. As you make changes to the files on Dropbox, the live site will be automatically updated accordingly. 

### What's included in the template?

The template includes an AngularJS-app, where you can add, delete and archive items.

**index.html** is just a wrapper for our app, and doesn't do anything else besides initialization.

**scripts/router.js** maps URLs to controllers and templates. This far there's nothing particularly interesting, but we'll be extending it during this tutorial to send variables for the controller.

**scripts/controller.js** contains `ListController`, which is responsible for logic of our app. In this tutorial we'll add a `service` that'll connect `ListController` to our back-end.

**templates/list.ngt** provides a template for our UI. You can read more about `.ngt`-extension from the previous part.

Alright, lets get started!

## Wiring up a backend 

In AngularJS, it's a good practice to wrap shared or complex functionality into so called *services*. One such service is `$resource` which provides an easy way to interact with backend APIs. In our case, we want to use Backlift's persistence API to save the list, so that your changes persist between page refreshes. Read more about Backlift persistence API from the [documentation](http://backlift.github.com/docs/persistence.html#the-persistence-api)

To start, add a `scripts/services.js`-file.

scripts/services.js:
	
    angular.module('ListServices', ['ngResource']).
    factory('ListItems', function($resource) {
        return $resource(
            '/backlift/data/itemlist/:id',  // URL template
            {id: '@id'},                    // default values
            {
                create: {method: 'POST'},
                update: {method: 'PUT'}
            }
        );
    });

This chunk of code creates a new angular resource, and connects it to an API with the given URL. The `:id` in the URL is a variable that will be filled in for a given list item. The second argument defines the default values used to create the URL. In this case `{id: '@id'}` means that the `id` property of the list item (the `@id`) will be mapped to the `:id` variable. If no `id` exists on the list item, that part of the URL will be empty. The third argument is a list of actions which will be added to the `ListItems` resource. By default angular adds the following actions to all resources:

	{ 'get':    {method:'GET'},
	  'save':   {method:'POST'},
	  'query':  {method:'GET', isArray:true},
	  'remove': {method:'DELETE'},
	  'delete': {method:'DELETE'} };

However, angular doesn't provide a PUT method by default, so we have added it above and called it `update`. We've also added a `create` action that does the same thing as angular's default `save`, but in our opinion makes its purpose clearer. Note that if an API method returns an array, its action must be flagged with `isArray: true`.

Using this resource, we're able to perform standard RESTful operations like so:

    ListItems.query()                   -> GET     itemlist
    ListItems.create(item)              -> POST    itemlist
    ListItems.update(item)              -> PUT     itemlist/:id
    ListItems.remove({id: '<item-id>'}) -> DELETE  itemlist/:id
  

> **Why not just call remove(item)?** Angular has some interesting features. One is that if you pass an object to a GET or DELETE action, any properties that are not defined as `@` parameters will be appended to the URL as query parameters. That means in our case `ListItems.remove({some: 'query'})` will result in a DELETE request sent to `/backlift/data/itemlist?some=query`. It also means that if we call `ListItems.remove(item)`, every propety of item, except the `id`, will end up in the URL as query parameters. This is usually not desirable.

Once we have defined our a `ListServices`-module, it has to be added as a dependency to the `listapp`-module in `router.js`.

Beginning of scripts/router.js:

	angular.module('listapp', ['ListServices']).
	...

This makes your `ListItems` resource available throughout your app.

Now you can Call the resource's methods, which result in HTTP requests, like this:

	function fn(ListItems) { 
		ListItems.create({text: 'A new item', archived: false}); 
		ListItems.query(function(items) { 
			console.log(items[0]);
		}); 
	}

To see this working in your browser, copy and paste the above function into your console, and then paste in the following code to run the method above:

	angular.element(window.document.body).injector().invoke(['ListItems', fn])

You should see the first item in your list printed to the console ('A new item'). Don't worry about the `….injector().invoke…` stuff for now. That's some magic we need in order to access our angular resource from the console.

> **What are the \_created and \_modified properties on the new item?** Backlift adds a few properties to every item it creates, including created and modified times, and an id if not specified. Generally Backlift prefixes its special properties with an \_. 

Now that we've created the service, let's go ahead and add `ListItems` to our controller so we can use it to persist our items.

scripts/controllers.js

    function ListController($scope, ListItems) {
    
        $scope.items = ListItems.query();
    
        $scope.addNewItem = function() {
            newitem = {
                'text':$scope.itemText,
                'deleted':false,
                'archived':false,
            }   
            newitem = ListItems.create(newitem);
            $scope.items.push(newitem);
            $scope.itemText = '';
        }

        $scope.deleteItem = function(item) {
            item.deleted = true;
            item.$update();
        }
    
        $scope.archiveItems = function() {
            for(var i in $scope.items) {
                var item = $scope.items[i];
                if(item.deleted && !item.archived) { 
                    item.archived = true;
                    item.$update();
                }
            }
        }   
    }

Lets see what's happening in here.

First, we use `ListItems.query()` to pull all items from Backlift API. Usually for an asynchronous call we would have a callback that'll get executed after the request is completed. However, `$resource` returns an empty array immediately and then later on populates it when the server responds. In the view, fetched items are rendered as soon as the array is updated. That's the magic of two-way data binding!

Second, in `addNewItem`, we use `ListItems.create(newitem)` to send new items to the server before we add them to the list. As mentioned above, Backlift will give each new item a random unique `id` automatically.

Finally, as you can see in `deleteItem`, we don't need to call a function on `ListItems` to update an item. Instead we can directly call `$update()` on the item. This is because items fetched with our `ListItems` resource actually inherit its methods. `$update` uses whatever data is currently stored in the object and sends it to the server. Because of this default `{id: '@id'}` value we supplied in `scripts/services.js`, angular will automatically add the `id` property of the item to the `:id` part of the URL. Since Backlift has already generated an `id` for each item, `item.$update()` results in an HTTP-call that looks something like `PUT /backlift/data/itemlist/123 "objectdata"`.

With the code we've written so far we can now store and retrieve items using a backend API. Obviously you don't have to use Backlift's server, any server that exposes the REST endpoints you define in your resource and that generates IDs will do.

## Generate new lists on the fly

Almost done! There's a neat UI for listing items and a backend-wrapper to save those to the server. Though, it would be cool if you could also easily create new lists. As last quick addition, let's modify `router.js`, `services.js` and `controllers.js` to create new lists on the fly.

scripts/router.js:

	var generateRandomId = function(){
		// Generates pseudo random new ID for list
		// NOTE: this isn't a very good random, but good enough for this
		return Math.random().toString(36).substring(9);
	}
	
	angular.module('listapp', ['ListServices']).
	config(function($routeProvider,$locationProvider) {
		$routeProvider.
			when('/', {
	      		redirectTo: '/'+generateRandomId()
			}).
			when('/:listid', {
				templateUrl: 'partials/list.ngt',
				controller: 'ListController'
			}).
			otherwise({
	      		redirectTo: '/'+generateRandomId()
			});
	});

Now the router gets more interesting. If user goes to base-url, she is redirected to an URL with a random string. If there is that random string (such as `www.myapp.com/asd123`), it is passed to the controller as a variable (`asd123` in this case).

Next, small modification to scripts/services.js:

    angular.module('ListServices',['ngResource']).
    factory('ListItems',function($resource){
        return $resource(
            '/backlift/data/:listid/:id',
            
			...

All we do, is replace the `itemlist`-part in URL with a `:listid`-variable.

Lastly in scripts/controllers.js:

    function ListController($scope, $routeParams, ListItems){

        $scope.url = 'https://'+window.location.host+'/'+$routeParams.listid;
        var ListItems = ListItems.bind({listid:$routeParams.listid});

        $scope.items = ListItems.query();
	
        ...

Two things happen in here. First, we add `$routeParams`-scope, which we use to access variables from `router.js`. Second, we bind `$resource` URL's `:listid`-parameter to the one we got from `router.js`. Now, all HTTP-calls made will be like `GET /backlift/data/asd123`.

With Backlift persistence API, if we make a call with non-existing list ID (so called *collection*), it will be created automatically. It makes this kind of approach really flexible and easy to prototype with.  

Finally, replace the angular template with the following code:

partials/list.ngt:

    <!-- .ngt stands for angular template -->
    <div class="row header">
        <div class="wrapper">

            <div id="header">
                <small>This is a list. It's live and online. Share it using the url below.</small>
                <h1><input type="text" value="{{url}}" /></h1>
            </div>

            <div id="add">
                <button href="" class="btn btn-delete right" ng-click="archiveItems()">Archive deleted</button>
                <form ng-submit="addNewItem()">
                    <input type="text" ng-model="itemText" placeholder="Add new item" />
                    <button href="" class="btn btn-primary" type="submit">Add</button>
                </form>
            </div>
        </div>
    </div>
    <div class="row">
        <div class="wrapper">
            <ul id="list">
                <li ng-repeat="item in items | filter:{archived:false} | orderBy:'_created':true" class="deleted-{{item.deleted}}">
                    {{item.text}}
                    <span class="date">{{item._created | date:'medium'}}</span>
                    <div class="action">
                        <a href="" ng-click="deleteItem(item)">del</a> 
                    </div>
                </li>
            </ul>
        </div>
    </div>

With that you can now create new lists on the fly!

You can download the final version from here:

<a target="top" href="https://www.backlift.com/backlift/dropbox/create?template=github.com/colevscode/backlift-angular-final&appname=quicklist"><button class="btn primary" style="font-size:larger;padding:5px"> Copy final version To Dropbox </button></a>

## Further ideas, DIY

Even if this tutorial is now finished, the list app definitely isn't. I strongly encourage you to explore both AngularJS and Backlift further and expand app's functionality. Some ideas for you to build:

- Add user management and make private lists. There's a Backlift authentication API supporting this, [check out documentation](http://backlift.github.com/docs/authorization.html#authorization).
- Support multiple lists, by adding more views and controllers.
- Extend HTML-vocabulary with directives to have <listitem>-element. This would really simplify our list.ngt-template

We hope this has been a valuable introduction to data persistence with Angular. Like always, you can add questions here or send us a note at feedback at backlift.com!