{{$ meta }}
title: angular or backbone-- what are startups using?
created: 2013-07-25
public: no
author: <a href="http://twitter.com/ColeVsCode">@colevscode</a>
disqusid: front-end-frameworks
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

Over the last few months there has been a lot of buzz about Angular.js. We are currently working on an overhaul of our front-end at Backlift, and have been itching to try out angular on a larger project, so we've been considering a switch. 

Don't get me wrong, we're huge backbone.js fans. Our entire site and most of our templates are written in backbone. But we've also written a few Angular.js tutorials and noticed that they are getting a lot of traffic recently. When our post on [Using AngularJS with a Back-end API](/entry/angular-tut2) hit the #1 spot on google for "angular backend" we figured we should pay a bit more attention.

Before making any decisions we wanted a bit more information. What other startups are using Angular? What kinds of companies are they, and what stacks are they using? We also wanted to know what other frontend technologies startups are using, like SASS or Coffeescript. To answer these questions we sent a poll out to hundreds of other startup founders and friends. We received over 30 responses and here are the results:

## What's your front-end stack?
 
{{$ with '{"name": "fw", "file": "/data/frameworks.csv"}' as data }}
	{{$ include /partials/d3bargraphs.html }}
{{$ endwith }}

We were struck by the fact that two thirds of companies that responded were using some kind of front-end MVC framework. Backbone was the most popular, and was also favored by larger startups, as determined by the size of their development teams. The majority of smaller startups, on the other hand, were either using Angular, or no MVC framework. It seems from this relatively small sampling that Backbone is currently the dominant MVC framework. On the other hand, perhaps Angular's double-binding and reduced boilerplate are compelling features for small teams with limited time.

We also asked what backend stack these companies were using. We had a theory that startups using a frontend MVC would be more likely to use a lightweight server-- just something to serve up data over a RESTful API. (That's what we're dong here at Backlift) Here's what we discovered:

{{$ with '{"name": "be", "file": "/data/backends.csv"}' as data }}
	{{$ include /partials/d3bargraphs.html }}
{{$ endwith }}

... Basically we saw an even split. The data didn't support our theory here.

Finally, we were curious which startups used front-end compiled languages like coffeescript, sass and less. We also wanted to know whether the presence of designers on the team had an impact. Here's what we found:

{{$ include /partials/drawpara.html }}


{{$ with '{"name": "des", "data":["designers", "styles", "scripts"]}' as data }}
	{{$ include /partials/paraset.html }}
{{$ endwith }}


Most startups that responded were using some kind of intermediate CSS language. SASS was the clear winner, with more than half of the startups using it alone. The startups without designers made up the majority of LESS users, while startups with at least one designer were especially unlikely to use plain CSS. The use of plain old javascript was slightly higher than coffeescript. 

## Conclusions

Overall most startups that responded to our survey were using some kind of front-end MVC framework, the most popular of which was Backbone.js. Also startups were more likely to use SASS or LESS than plain old CSS. Beyond that, it was difficult to draw sweeping conclusions. However, we may have missed something, so we're including the full dataset [here](/data/data.csv), along with a <a href="" id="kitchensink">kitchen-sink visualization of the data</a>. (Tufte, please forgive us)

{{$ with '{"name": "all", "style":"display: none; margin-bottom: 10px", "data":["framework", "devs", "designers", "backend", "scripts"]}' as data }}
	{{$ include /partials/paraset.html }}
{{$ endwith }}

<script>
$('#kitchensink').click(function(ev) {
	ev.preventDefault();
	$("#visall svg").show("slow");
});
</script>


We'd love to hear what you're using. Feel free to hop over to [this survey]() and send us your data. If the results reveal a different trend, we'll happily post an update. Also feel free to tell us what front-end stack you're using by posting in the [Hacker News comments]().
