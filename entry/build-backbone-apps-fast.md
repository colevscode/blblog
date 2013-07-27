{{$ meta }}
title: build backbone.js apps fast with backlift
created: 2013-06-18
public: no
author: <a href="http://twitter.com/ColeVsCode">@colevscode</a>
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

Howdy all. Cole, lead dev at [backlift.com](https://backlift.com), here. This is a tutorial to help you give a quick overview of Backlift, and show you how to create a rather complex backbone.js app from scratch. In the interest of brevity, this tutorial won't get into user accounts or security, however this functionality is available within Backlift. Please see the [Backlift docs](http://backlift.github.com/docs/) for more info.

Let's get started!

## The Setup

First go sign in at [backlift.com](https://backlift.com) with Dropbox. Don't use dropbox? Um... crap. Please talk to me after class[*](#nodropbox). 

For everyone else, once you've signed in click the "create app" button, type an app name in the box like "myfirstapp", leave the "hello world" template selected and click "Create".

(photo) 

Now we do some thinking... and voila, a new app is born. If you click the "view app" link it will take you to the app's public website. 

## The Workflow

Just as a quick test, lets make a change and make sure things are working correctly. Navigate to your new app's folder within your Dropbox. It should be located somewhere like `blabla/Dropbox/Apps/Backlift/mynewapp`. Once you've found it, open up the index.html file with a text editor. Scroll half way down the file and find the line that says 

    <h1> Hello World! </h1>

Now change it to something new like

    <h1> Hello Universe! </h1>

Once you save it, if the app is still open in your browser it should refresh in a few seconds, offering a more grandiose salutation.

## Setting up backbone.js

First things first, we've got to grab backbone.js and a few other libraries. There are Backlift templates that come with all this pre-packaged, but for the sake of pedagogy, I'm going to do this manually. Create a new folder called "libraries" in your app's root folder. Then copy these files into that folder:

* [jQuery](http://code.jquery.com/jquery-1.8.3.js)
* [json2.js](//cdnjs.cloudflare.com/ajax/libs/json2/20121008/json2.js)
* [underscore.js](http://underscorejs.org/underscore.js)
* [backbone.js](http://backbonejs.org/backbone.js)

Now copy the bootstrap.css and backlift-reloader.js files into the libraries folder.

Finally, add a style.css file to the root of the app folder containing this code:

    blockquote {
        margin-top: 150px;
    }

At this point, your app folder should look like this:

<img src="/images/tut1-1.png">

Now let's setup a simple index.html file that links in our javascript and css files. Open the index.html file, delete all the content and replace it with this:

    <!DOCTYPE html>
    <html>
      <head>
        <link href='/libraries/bootstrap.css' type='text/css'>
        <link href='/libraries/style.css' type='text/css'>
      </head>
      <body>

        <div class="container">
          <blockquote>
            <p>
              I postpone death by living, <br>
              by suffering, by error, by risking, <br> 
              by giving, by losing.
            </p>
            <small>Anais Nin</small>
          </blockquote>
        </div>

        <script src='/libraries/jquery-1.8.3.js'></script>
        <script src='/libraries/json2.js'></script>
        <script src='/libraries/underscore.js'></script>
        <script src='/libraries/backbone.js'></script>
        <script src='/libraries/backlift-reloader.js'></script>
      </body>
    </html>

After a moment's contemplation the app should refresh in your browser.

## A Touch of Magic

Hard coding a bunch of script and style tags is so passe. We're going to tell Backlift what libraries we're using so we can manage them in one place, rather than keeping separate lists of &lt;script&gt; tags in all our html files. To do that, let's create a config.yml file and place it in our app's root folder. The file should contain the following:

    scripts: 
        - /libraries/jquery-1.8.3.js
        - /libraries/json2.js
        - /libraries/underscore.js
        - /libraries/backbone.js
        - /**/*.js

    styles: 
        - /libraries/bootstrap.css
        - /**/*.css

The order is important, because it will determine the order the files are linked into our index.html. In this case, since backbone depends on jquery, json2 and underscore, we've ordered them so that those dependencies are loaded first. The `/**/*.js` line will add any other javascript files discovered in the app's folder to the project. (Note that we didn't add /style.css, the `/**/*.css` line catches it.)

To take advantage of the config.yml file, replace the script and style tags in our index.html file with the special template variables {{scripts}} and {{styles}}. Backlift will fill in the list of files. 

Make your index.html file look like this: 

    <!DOCTYPE html>
    <html>
      <head>
        {{ styles }}  <!-- will be replaced with styles from config.yml -->
      </head>

      <body>

        <div class="container">
          <blockquote>
            <p>
              I postpone death by living, <br>
              by suffering, by error, by risking, <br> 
              by giving, by losing.
            </p>
            <small>Anais Nin</small>
          </blockquote>
        </div>

        {{ scripts }}  <!-- will be replaced with scripts from config.yml -->
      </body>
    </html>

What a piece of artwork.

## Project: A Poor Man's Pinterest

For the rest of this post I'll be walking you through the steps required to build an gallery app that showcases photos from your Dropbox. It should look something like this:

(sketch)

We'll start by roughing out the app with some static HTML, and then swap it out with dynamic stuff later. 

First replace the blockquote in the index.html with the following code:

        <div class="container">
          <h1> My Gallery </h1>
          <div class="gallery">
            <img src="/photos/sup.gif">
            <img src="/photos/sup.gif">
            <img src="/photos/sup.gif">
            <img src="/photos/sup.gif">
            <img src="/photos/sup.gif">
            <img src="/photos/sup.gif">
          </div>
        </div>   

Then delete the contents of style.css and replace it with:

    h1 {
        margin-top: 1em;
    }

    .gallery img {
        max-width: 300px;
        margin: 10px 10px 0 0;
    }

Finally, create a `photos` folder at the root of the app and move `sup.gif` into it. Once the page refreshes, we should have a gallery of cats. Woopie!

## Hard Core Backbonin'

Ok, enough pillow talk. Let's make our gallery of photos dynamic, so that we can add photos without changing our HTML. To do this we'll use a Backbone.Collection to keep track of our photos, and a Backbone.View to render the photos in our collection.

Create an `app` folder and create two files in that folder, one called `gallery.js` and the other called `main.js`. Your project should now look something like this:

<img src="/images/tut1-3.png">

Add the following content to your new `gallery.js` file:

    App.GalleryView = Backbone.View.extend({
      initialize: function() {
        this.listenTo(this.collection, 'add', this.renderPhoto);
      },
      renderPhoto: function(photo) {
        this.$el.append("<img src='/photos/sup.gif'>");
        return this;
      }
    });


And add this to your `main.js` file:

    // create the global App namespace
    App = {};

    // main block, execution starts here
    $(function() {
      App.photos = new Backbone.Collection();
      App.gallery = new App.GalleryView({ 
        collection: App.photos,
        el: "#gallery"
      });
    });

Let's break this down a bit. In `gallery.js` we've defined a GalleryView that will be used to render our photo gallery. The block at the bottom of `main.js` is the starting point of execution for our app, and is evaluated after the browser has finished loading. 

The main block begins by creating a photos collection and passing it to a new GalleryView. We also set the GelleryView's `el` attribute to the element with id "gallery", which is the &lt;div&gt; where our photos will end up in `index.html`.  During initialization, the GalleryView starts listening to the photos collection for add events. Each time an add event is triggered, GalleryView responds by calling `renderPhoto`, which appends a new image to its `el` node-- the "gallery" &lt;div&gt; we passed it.

We must do one last thing before our app will work. Edit your `config.yml` file so that the `scripts` block includes the `main.js` file, like so:

    scripts: 
        - /libraries/jquery-1.8.3.js
        - /libraries/json2.js
        - /libraries/underscore.js
        - /libraries/backbone.js
        - /app/main.js   # <-- new
        - /**/*.js

This is required because, at the top of `main.js` we create the `App` global namespace. We must ensure that `main.js` is loaded before the other scripts that use the `App` variable.

To test out the new code, first delete each of the &lt;img&gt; tags in the index.html file. Then open the browser's console and type:

    App.photos.add()

Each time you type this into the console, you should see a new cat photo added to the gallery.

## Fetching Photos from Dropbox

So far we've only worked with hard-coded images. To create a real gallery, however, we need a way to display arbitrary images. Backlift offers an API for listing the contents of a folder within our app, and we can use Backbone to access that API. Rather than using the default Backbone.Collection type, we'll create our own Collection that uses the Backlift table of contents API. 

At the top of gallery.js, create a new subclass of Backbone.Collection with the url, `/backlift/toc/photos/*`. Your gallery.js file should look like this:

    App.PhotosCollection = Backbone.Collection.extend({
      url: "/backlift/toc/photos/*"
    });
    
    App.GalleryView = Backbone.View.extend({
    ...

Now in the main block of `main.js`, initialize the App.photos collection using the App.PhotosCollection class. Then call `fetch()` on the collection to load the table of contents data:

    // main block, execution starts here
    $(function() {
      App.photos = new App.PhotosCollection();
      App.photos.fetch();
      App.gallery = new App.GalleryView({ 
        collection: App.photos,
        el: "#gallery"
      });
    });

When the page refreshes, our app will fetch the photos collection, which issues an HTTP GET request to the collection's url. This request will return a table of contents for all files in our app's folder that match the search string "/photos/*". 

At this point, there's no way to know if our query worked just by looking at the webpage in the browser. However we can use our browser's console to see if the fetch request was successful. If you use a webkit browser like Chrome or Safari, bring up the developer console, click the gear in the lower right corner and ensure that "Log XMLHttpRequests" is checked. 

<img src="/images/tut1-4.png">

Then reopen the console and reload the app. You should see a line that looks something like

    XHR finished loading: "https://myfirstapp-gs75g.backliftapp.com/backlift/toc/photos/*". 

If you click the link, it will highlight the request in the network tab. Clicking the path will reveal the response:

<img src="/images/tut1-5.png">

From here we can see that backlift has returned a JSON array containing a single object with url, file, created and modified attributes that match the sup.gif file in our photos folder. We can use this data, especially the URL attribute, to populate our gallery with photos. 

Change your App.GalleryView class in gallery.js to match this:

    App.GalleryView = Backbone.View.extend({
      initialize: function() {
        this.listenTo(this.collection, 'reset', this.render);   // <-- new 3
      },
      renderPhoto: function(photo) {
        this.$el.append("<img src="+photo.get('url')+">");      // <-- new 1
        return this;
      },
      render: function() {                                      // <-- new 2
        this.$el.empty();
        this.collection.each(this.renderPhoto, this);
        return this;
      }
    });

We've made three changes above. First, in `renderPhoto`, we've inserted the URL attribute into the &lt;img&gt; tag that we render for each photo. Second we've added a `render` function that iterates through the collection rendering each photo in the gallery. Finally, in `initialize` we've replaced the 'add' event listener with a 'reset' event listener that will be triggered when backbone.js successfully fetches a collection.

At this point our gallery should display any photos we drop into the photos folder. For example, if we add this [photo](/images/tut1-6.jpg), our gallery should look like this:

<img src="/images/tut1-7.png">

## Comments, comments, comments

For the last part we'll add comments. Each photo will have a list of comments below it. For now we won't worry about user accounts, we'll just make it possible for anyone who visits the gallery to add an anonymous comment.

Let's add a new collection to our app to hold the comments. Your `gallery.js` file should look like this:

    App.PhotosCollection = Backbone.Collection.extend({
      url: "/backlift/toc/photos/*"
    });

    App.CommentsCollection = Backbone.Collection.extend({  // <-- new
      url: "/backliftapp/comments"
    });

    ...

Note that we're using Backlift to persist the comments collection. We don't need to set up anything special to do this-- when we start to use the new collection Backlift will just do the right thing. 

In order to display photos with comments attached, we'll need to render a more complex photo view. We can do this by using a javascript templating library like [Handlebars.js](handlebarsjs.com). Save the [handlebars script](http://cloud.github.com/downloads/wycats/handlebars.js/handlebars-1.0.rc.1.js) to the `libraries` folder and add this line to `config.yml`:

    scripts: 
        - /libraries/jquery-1.8.3.js
        - /libraries/json2.js
        - /libraries/underscore.js
        - /libraries/backbone.js
        - /libraries/handlebars-1.0.rc.1.js  # <-- new
        - /app/main.js 
        - /**/*.js

Now let's create a new file called `photo.handlebars` in our `app` folder with this content:

    <div class="photo">
      <img src="{{photo.url}}">
      <ul class="comments">
        {{#each comments}}
          <li class="comment">{{msg}}</li>
        {{/each}}
      </ul>
    </div>

We should probably add a few styles so our photo views don't look like crap. Replace the contents of `style.css` with:

    h1 {
        margin-top: 1em;
    }

    #gallery img {
        max-width: 290px;
        padding: 5px;
    }

    #gallery div {
        display: inline-block;
        float: left;
        border: 1px solid #aaa;
        border-radius: 5px;
        margin: 10px 10px 0 0;
        background-color: #EEE;
    }

    #gallery ul {
        padding: 5px;
        margin: 0;
        list-style-type:none;
    }

    #gallery li.comment {
        padding: 3px;
        margin-bottom: 5px;
        border-radius: 5px;
        background-color: #FFF;
    }


Now lets change the `GalleryView.renderPhoto` function to use the new template:

      renderPhoto: function(photo) {
        var comments = _.where(this.options.comments.toJSON(), {photo: photo.get('file')});
        var data = {
          photo: photo.toJSON(),
          comments: comments
        }
        this.$el.append(Handlebars.templates.photo(data)); 
        return this;
      },

In the above function, we first get a list of all comments that match the current photo-- any comment with a `photo` property that matches the current photo's `file` property is included. Then we package up the current photo along with its comments into a `data` object and use it to render our template. (Backlift automatically compiles our `photo.handlebars` file and makes it available as the `photos()` function on `Handlebars.templates`.)

In order to see new comments, we can add another listener in the `initialize` method:

      initialize: function() {
        this.listenTo(this.collection, 'reset', this.render); 
        this.listenTo(this.options.comments, 'add', this.render);   // <-- new
      }, 

Finally we have to instantiate a new comments collection and pass it to `App.gallery` when it's created. Open up `main.js` and add these lines:

    $(function() {
      App.photos = new App.PhotosCollection();
      App.photos.fetch();
      App.comments = new App.CommentsCollection();    // <-- new
      App.gallery = new App.GalleryView({ 
        collection: App.photos,
        el: "#gallery",
        comments: App.comments                        // <-- new
      });
    });

Now if we open the console and type...

    App.comments.create({photo: "sup.gif", msg: "hey there!"});

... We should see the new comment appear under the `sup.gif` photo.

<img src="/images/tut1-8.png">

## Intermezzo: Fetching data at page load

Unfortunately at this point if you reload the page, the comments disappear. That's because we haven't called `App.comments.fetch()` in `main.js`, like we do with `App.photos`. However it's not as straight forward as simply calling `fetch()` twice. With photos, we're using the 'reset' event to trigger the gallery render once the photos have been fetched. Using the same technique for comments would cause the gallery to render twice on page load.

There's another problem with this approach. Fetching data at page load slows down the initial page render, and makes the website feel slow. The right way to do this is, if you were building your own server, would be to have the server inject some javascript at the bottom of the page with initial data for those collections. Then we could render the gallery right away instead of waiting for several fetch responses. Backlift is working on an API that will inject initial javascript payloads at the bottom of our `index.html` file, but it's not ready yet. Until then, we need a work-around.

Replace the `main.js` file with the following code:

    // create the global App namespace
    App = {};

    function withCollections(collections, fn) {
    	_.each(collections, function(col) {
        col.fetch({
          success: function() {
            col.fetched = true;
            var done = _.every(collections, function(item) {
              return item.fetched == true;
            });
            if (done) {
              fn.call();
            }
          }
        })
      });
    }

    $(function() {
      App.photos = new App.PhotosCollection();
      App.comments = new App.CommentsCollection();
      withCollections([App.photos, App.comments], function() {
        App.gallery = new App.GalleryView({ 
          collection: App.photos,
          el: "#gallery",
          comments: App.comments
        });
        App.gallery.render();
      })
    });

The `withCollections` function will wait until all collections have been loaded before calling its function argument. Because we're now calling `App.gallery.render()` from `main.js`, we can remove the 'reset' event listener in `gallery.js`, giving us this:

    ...

    App.GalleryView = Backbone.View.extend({
      initialize: function() {
        this.listenTo(this.options.comments, 'add', this.render);
      },

    ...

(We're leaving the 'add' event listener for later)

Now when you reload the page you should see the comments that you previously added.


## Final Step: Add Comment Form

For the piece de resistance we're going to add a form to our photo views so that onlookers can contribute their two cents to our photos. We'll start by adding a form to our `photos.handlebars` template:

    <div class="photo">
      <img src="{{photo.url}}">
      <ul class="comments">
        {{#each comments}}
          <li class="comment">{{msg}}</li>
        {{/each}}
        <li>                                                   <!-- new -->
          <form class="form-inline" id="{{photo.file}}">
            <input type="text" id="msg">
            <input type="submit" class="btn">
          </form>
        </li>          
      </ul>
    </div>

Then let's add an events property at the bottom of our `GalleryView` definition that handles the form submit event:

    App.GalleryView = Backbone.View.extend({

      ...

      events: {
        "submit": function(ev) {
          ev.preventDefault();
          this.options.comments.create({
            photo: $(ev.target).attr("id"),
            msg: $(ev.target).find("#msg").val()
          });
        }
      }
    });

Finally add one more style to the bottom of `style.css`:

    #gallery form {
        padding: 0;
        margin: 0;
    }

Reload and voilla! You can start adding comments!

<img src="/images/tut1-9.png">

## Conclusion

I hope this tutorial has given you a taste of how you can quickly prototype sophisticated apps with backbone.js and Backlift. This example leaves a lot of room for improvement, specifically with respect to the gallery layout and interaction. (Submitting a comment shouldn't trigger a full gallery refresh, for example) A more complete version of the gallery app will be added to the Backlift templates soon. Thanks for trying out backlift, and let me know if you have any questions at cole (at) backlift (dot) com! Cheers!

