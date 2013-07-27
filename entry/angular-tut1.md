{{$ meta }}
title: angular.js views and controllers
created: 2013-03-25
public: yes
author: <a href="http://twitter.com/laurihy">@laurihy</a>
disqusid: be52b328-e7e2-4342-ba43-d9f3e6ba73bf
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

In this walkthrough, we'll get started with AngularJS, a popular JavaScript framework for web applications. To demonstrate some of the best features of AngularJS, we'll build an app that generates disposable lists with unique URLs - imagine Pastebin for lists. It's more exciting than just the good-old todo app, and would be a great fit for grocery lists or even a temporary (asyncronous) chat. 

Check out the final version here: <https://angularfinal-ep2wx.backliftapp.com>

{{$ break }}

This tutorial will be very hands-on so we won't go too much into AngularJS internals or details. You should consider this a quick peek into AngularJS and also check out the [official AngularJS guide](http://docs.angularjs.org/guide/). In about 20 minutes, you'll have a good overview on how AngularJS-apps are built, what are their strenghts and why you should be interested.

### Why AngularJS

AngularJS reduces a lot of the boilerplate code typical of modern web applications. Normally you might need to write a lot of code just to keep the UI and models in sync, but Angular's *two way data-binding* takes care of that for you. Also, by using *directives* you can actually extend HTML-vocabulary to include new elements. For more AngularJS-inspiration, check out [these 5 awesome AngularJS features from TutsPlus](http://net.tutsplus.com/tutorials/javascript-ajax/5-awesome-angularjs-features/)

### Why Backlift

In this tutorial, we'll use Backlift to host the app and provide a RESTful API for saving lists. AngularJS and Backlift go very well together, since AngularJS makes it easy to work with backend API's (as we'll see later) and with Backlift you can dive right into the front-end code without writing a back-end first.

## Setup Backlift

To kick off everything, use this link to setup an initial project template. All you need, is a Dropbox account and the Dropbox client installed on your computer. 

<p><a target="top" href="https://www.backlift.com/backlift/dropbox/create?template=github.com/laurihy/AngularTutorial1-Setup&appname=angulartemplate"><button class="btn btn-primary btn-large" style="font-size:larger"> Copy Part One-Setup to Dropbox </button></a></p>

This creates a new folder in your Dropbox (something like /Apps/Backlift/newAppName) containing `index.html` and the AngularJS-library. Also, the site is immediately live. Just click the "view app" link in Backlift to view it.

Any changes you make to the files on Dropbox will automatically sync to the live site. To check this out, open `index.html` in a text editor and make some changes to the file. Now, if you wait a few seconds, that page will automatically refresh when Backlift detects the changes.

> **What are `{{$ scripts}}` and `{{$ styles}}` in `index.html`?** They are variables provided by Backlift and are used to include all necessary scripts and styles defined in `config.yml`. As you can see by viewing app's source in browser, they get replaced by a bunch of `<script>` and `<style>`-tags and help to reduce boilerplate in that file. For more info about the `config.yml` file and Backlift's server variables, check out the [Backlift docs](http://backlift.github.com/docs/basics.html#backlift-variables-and-the-configyml-file).

## Structure of our final app

Before diving into the code, let's go briefly over the components we will build during this walkthrough.

**Index.html** includes all scripts and styles we're using, just as a regular .html-file would. It will also initialize our AngularJS-app and acts as a container for other templates.

**Templates** are snippets of HTML, which contain parts of the UI of our application. In this tutorial we'll use only one, but in a bigger app it would make sense to separate different, re-usable UI components into separate templates.

**Controllers** handle most of the complex logic related to data and templates. The core of a controller is the `$scope` object, which injects data into a template. It binds data in both ways, meaning that changes are propagated from controllers to views and vice versa. It's really handy, and we'll get into that a bit later.

**Router** maps URLs to certain templates and controllers. Routers make deep links possible.

**Services** are wrappers for functionality in AngularJS. We will implement a `resource-service`, which provides a nice wrapper for the Backlift API that we use to persist items in the lists.

Generally, Angular follows the MVC-pattern, which stands for Model-View-Controller. While there's a [theoretical foundation](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), in practice it means that data, business logic and it's presentation are separated into their own components. This approach makes it easier to modify, for example, the UI (presentation) without worrying much about underlying logic (controllers).

## Hello AngularJS

Let's spice-up our boring static app with some AngularJS. As a quick teaser, add `<span ng-bind="1+1">Loading</span>` inside the `<body>` and append `ng-app` to `<html>`-tag.

index.html:

	<!doctype html>
	<html ng-app>
		<head>
			{{$ scripts}}
			{{$ styles}}
			<title>Quicklist</title>
		</head>
		<body>
			<span ng-bind="1+1">Loading</span>
		</body>
	</html>

Both are AngularJS extensions to HTML. `ng-app` initializes an AngularJS-app and without it anything else we do in AngularJS wouldn't work. `ng-bind` is used to bind any expression or variable as the content of the tag. In this case, when you have your app initiated properly with `ng-app`, content of `span` should be "2". If there's an error, `ng-bind` doesn't get processed and `span` reads just "Loading".

Without further ado, let's add some structure to our app.

### Controllers

Storing application logic inside few `ng-bind`s doesn't get us very far, so let's introduce a *controller*. To do this, modify `index.html` and add `scripts/controllers.js` file.

scripts/controllers.js:

	function ListController($scope){
		$scope.name = "Controller";
	}

In `ListController` we use `$scope` variable, which carries data between the controller and the view (in this case `index.html`). Variables added to `$scope` are accessible inside a view and, on the other hand, variables defined in the view can be accessed from `$scope`. Let's see how we can tie it into view by modifying `index.html`.

index.html:
	
	<!doctype html>
	<html ng-app>
		<head>
			{{$ scripts}}
			{{$ styles}}
			<title>Quicklist</title>
		</head>
		<body ng-controller="ListController">
			<h1>Hello from <span ng-bind="name">...</span></h1>
		</body>
	</html>

Here we attach our controller to `<body>` using `ng-controller`. Variable `ng-bind="name"` refers to `$scope.name` used in `ListController`. As variables change in the controller, they will also change in the rendered view. Try changing `$scope.name` in `controllers.js`. Once Backlift refreshes the page, you should see a your customized greeting.

> **What's the dollar sign?** $ is used to prefix AngularJS's own methods and to avoid clashes with other methods and variables. Don't worry, it has nothing to do with jQuery's $-object.

### Add a router and separate template

One of the great things in web applications is being able to link into some specific part of the app. This is called *deep linking* and is achieved through mapping URLs to different views and variables. We do this by creating a router, so add a new file `scripts/router.js`.

scripts/router.js:
	
	angular.module('listapp', []).
		config(function($routeProvider) {
		$routeProvider.
			when('/', {
				templateUrl: 'partials/list.ngt',
				controller: 'ListController'
			}).
			otherwise({
				redirectTo: '/'
		});
	});

In the `$routeProvider` we map URL patterns to certain templates and controllers. This way we can dynamically change and alter things as the URL changes without needing to actually refresh the page. For now we don't have anything too exciting, just a blank `/` (as in www.myapp.com/) and everything else is mapped to that same route. On the other hand, there could be for example `/create`, where we would use a different template with a form. 

Next, we will create a file, `partials/list.ngt`, which contains our template. This is the template referred to by the `templateURL` property of our router above.

partials/list.ngt:

	<div class="wrapper">
		<h1>Hello from <span ng-bind="name">...</span></h1>
	</div>

Finally, we'll move the UI away from `index.html` to the template we just created. To use a template, we will add the `<ng-view>`-tag to `index.html`. Also, we'll provide `ng-app` the module `listapp`, which is the name of our router.

New index.html:

	<!doctype html>
	<html ng-app="listapp">
		<head>
			{{$ scripts}}
			{{$ styles}}
			<title>Quicklist</title>
		</head>
		<body>
			<ng-view></ng-view>
		</body>
	</html>
	

At this point, our website should look the same as before. However, we've restructured it to use a router and a template. The contents of `list.ngt` should be rendered inside `<ng-view>`.

> **NOTE**: in Angular you can use either `ng-bind` or `{{ }}`-notation to render expressions and variables. Usually in AngularJS, your templates would be just regular .html-files, but because Backlift processes `{{ }}`-instances internally as variables in .html-files, we use different file extensions. The extension you use doesn't really matter and could as well be .asd or .cat (.ngt stands for aNGular-Template).

## Using directives to loop over a list of data

*Directives* are used to teach HTML new tricks. You can either add new functionality with new attributes, or even define app-specific tags. Imagine just writing `<todo>` instead of an ugly bunch of nested `<div>`s. While you are free to define your own directives, there's also many readily available in AngularJS, such as `ng-controller` we've already used. In this section, we'll use `ng-repeat` to loop over a list of items.

Let's add a list to our `$scope` in `controllers.js` and then modify the template in `list.ngt`.

scripts/controllers.js:

	function ListController($scope){
		$scope.items = [
			{text:'Hello'},
			{text:'From'},
			{text:'ListController'}
		]
	}

partials/list.ngt:	
	
	<div class="wrapper">
		<ul>
			<li ng-repeat="item in items">{{item.text}}</li>
		</ul>
	</div>

We repeat `<li>`s for every element in items, and can also refer to each item's properties, such as `item.text`. Also, instead of using `ng-bind` we render the variable using `{{item.text}}`. This is mostly just a matter of preference, I happen to like the looks of the curly braces.

## Adding new items with two-way data binding

Let's see how easy it's to add new items to our list. First, there has to be a form in `list.ngt`.

partials/list.ngt:


	<div class="wrapper">
		<form ng-submit="addNewItem()">
			<input type="text" ng-model="itemText" placeholder="Add item" />
			<button type="submit" class="btn">Add</button>
		</form>
		
		<ul>
			<li ng-repeat="item in items">{{item.text}}</li>
		</ul>
	</div>
	

There are 2 interesting things here. `ng-submit="addNewItem()"` adds an action to be performed when the form is submitted. In a minute, we define that method in `controllers.js` and add it to our `$scope`. `ng-model="itemText"` binds a variable with that name to `<input>`s value. That's also something we can access in `controllers.js` by using `$scope`.

Now, let's add the corresponding functionality into `ListController`.

scripts/controllers.js:

	function ListController($scope){
	
	    $scope.items = [
	        {text:'Hello'}
	    ];
	
	    $scope.addNewItem = function(){
	        var newItem = {text: $scope.itemText}
	        $scope.items.push(newItem);
	
	        // clear the form text, 2-way binding \o/
	        $scope.itemText = '';
	    }
	}

After user submits the form, `$scope.addNewItem()` is called in `controllers.js`, value of the input fields is read from `$scope.itemText` and new item is added to `$scope.items`. The added item is automatically inserted to the view, just try it out yourself.

Here we witness the very best part of AngularJS: *two way data-binding*. Variables between the view and the controller are automatically synchronized on both ways, using the `$scope`-object. If a variable such as `itemText` changes, we can always access it in `ListController`. On the other hand, whenever we change variable in `ListController`, it's changed in the view as well. For example, when we add new item to `$scope.items`, `ng-repeat` in `list.ngt` is immediately updated with new item. 

### Toggle styles with two way data-binding

Using two way data-binding, we can toggle styles in our view. Let's implement a delete operation so, that each item can be deleted, which in practice toggles a class for each `<li>`.

First, add a clickable `<a>` into `list.ngt`, with directive `ng-click="deleteItem(item)"`.

partials/list.ngt:

	<div class="wrapper">
	    <form ng-submit="addNewItem()">
	        <input type="text" ng-model="itemText" placeholder="Add item" />
	        <button type="submit" class="btn">Add</button>
	    </form>
	
	    <style>
	        .deleted-true {
	            opacity: 0.2;
	        }
	    </style>
	
	    <ul>
	        <li ng-repeat="item in items" class="deleted-{{item.deleted}}">
	            {{item.text}} 
	            <a href="" ng-click="deleteItem(item)">(del)</a>
	        </li>
	    </ul>
	</div>

In the same way as in our previous example, `deleteItem()` refers to a method in current `$scope`, defined in `controller.js`. `<li>` has a class, which is either `deleted-true` or `deleted-false` depending on `deleted`-value of each item.
	
scripts/controllers.js:

	function ListController($scope){
	
		$scope.items = [
			{text:'Hello', deleted: false}
		];
	
		$scope.addNewItem = function(){
			newItem = {
				text: $scope.itemText,
				deleted: false
			}
			$scope.items.push(newItem);
	
			// clear the form text, 2-way binding \o/
			$scope.itemText = '';
		}
	
		$scope.deleteItem = function(item){
			item.deleted = true;
		}
	}

When `$scope.deleteItem()` is triggered, `item.deleted` is marked as true. When this happens, also `<li class="deleted-{{item.deleted}}">` changes in the view along with displayed style.

### Filtering a list and a task for you to do

In addition to just looping through items in array with `ng-repeat`, it's also easy to add dynamic filters and sorting. For example, if we want to filter out deleted items and sort them alphabetically, we can do it like this:

	<li ng-repeat="item in items | filter:{deleted:false} | orderBy:'text':true">

For our app, we want to add a second flag for items that hides them completely from the list. Users first mark items as completed, and then when they are sure they're not needed anymore, there's a separate `archive` button that filters all completed items from the list.

Why not try to implement that yourself?

1. Add an `archive` button to the template and bind it to a function
2. Write a function on the controller that loops through all items, finds those marked `deleted` and applies the `archived` flag
3. Modify the filter in the template to only display items that haven't been `archived`.

The final version including the changes above can be downloaded from here:

<a target="top" href="https://www.backlift.com/backlift/dropbox/create?template=github.com/laurihy/AngularTutorial1-Final&appname=angularlist"><button class="btn primary" style="font-size:larger;padding:5px"> Copy Part One-Final to Dropbox </button></a>

## To be continued... 

That's all for now, but there's more coming. In Part Two we'll show you how to wire up a back-end by implementing Angular's `resource-service` interface. In addition we'll talk about how to generate a new empty list on the fly when a user visits the landing page (`/`).  We hope this has been valuable! Please leave feedback here or shoot us an email at feedback (at) backlift.com. You can also follow us at @backliftapp on twitter!