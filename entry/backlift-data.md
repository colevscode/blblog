{{$ meta }}
title: getting started with backlift data
created: 2013-04-15
public: yes
author: <a href="http://twitter.com/ColeVsCode">@colevscode</a>
disqusid: 9048f210-7a62-4c40-9e7a-54d7a5d2b6ad
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

A few devs have been asking about how to get started with the Backlift data API. Our documentation on data persistence can be found [here](http://backlift.github.io/docs/persistence.html) and [here](http://backlift.github.io/docs/api.html#data-persistence). However, I thought a quick tutorial would make for a better introduction.

Probably the easiest way to get started with data is to open the console and kick off a few ajax requests. {{$ break }} (If you're not familiar with the console, check out [this stack overflow question](http://stackoverflow.com/questions/4743730/what-is-console-log-and-how-do-i-use-it). If you don't know ajax, take a look at [this overview](http://learn.jquery.com/ajax/).) The helloworld template comes bundled with jQuery, so start by creating a new helloworld app.

<p>
<a target="top" href='https://www.backlift.com/backlift/dropbox/create?template=helloworld&appname=helloworld&description=A starter template that includes jQuery, underscore, Twitter Bootstrap, and the Backlift admin.'><button class='btn btn-large btn-primary' style='font-size:larger'>Create a Hello World app</button></a>
</p>

Now, say you want to create a new 'monkeys' collection and add one monkey named 'Frank'. You can do it all at once with a POST request:

    $.post("/backlift/data/monkeys", {name:"Frank"});

If you look in your admin data browser, you'll see a new monkeys collection including Frank. Note that you didn't have to create the monkeys collection first. When Backlift receives a POST request for a collection that doesn't exist yet, it creates the collection on the fly.

In order to retrieve your data, you can use a GET request. Something like this:

    $.get("/backlift/data/monkeys", function(data) {
        console.log(data);
    });

You'll see an array printed out to the console with one monkey in it. 

Let's say you want to get one monkey at a time, instead of an array containing all the monkeys. Well, first we need to know the monkey's ID. Let's create a new monkey and assign it's ID to a global variable:

    $.post("/backlift/data/monkeys", {name:"Wilma"}, function(data) {
        window.monkeyID = data.id;
    });

Now in the console you can type window.monkeyID, and you'll see the unique identifier that Backlift has assigned to Wilma. Armed with this information, you can now retrieve just Wilma's document by typing:

    $.get("/backlift/data/monkeys/"+window.monkeyID, function(data) {
        console.log(data);
    });
 
What if you want to specify your own ID, rather than have Backlift assign one for you? No problem! Just add an `id` field to the object when you create it like so:

    $.post("/backlift/data/monkeys", {name:"George", id: 1});

What if you need to delete a monkey? First wait until there are no animal rights activists nearby, then type:

    $.ajax({type: "delete", url: "/backlift/data/monkeys/1"});

This will delete George, the last monkey we created with an ID of 1. Our evil plan to take over the world with a monkey replicator is progressing nicely.

In summary, creating and deleting Backlift objects follows a simple pattern:

    // Creates and object
    $.post("/backlift/data/<collection>", {...});

    // Reads and object
    $.get("/backlift/data/<collection>/<id>", function(data) {...});

    // Updates and object
    $.ajax({type: "put", url:"/backlift/data/<collection>/<id>"}, {...});

    // Deletes and object
    $.ajax({type: "delete", url:"/backlift/data/<collection>/<id>"});

***Extra Credit***: You may have noticed that the objects returned include several additional properties, such as `_owner` and `_id`. These properties are added by Backlift and are used to manage the object. More info about these special properties can be found [here](http://backlift.github.io/docs/persistence.html#special-object-attributes).