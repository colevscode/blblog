{{$ meta }}
title: "angular or backbone: what are startups using?"
created: 2013-07-25
public: no
author: <a href="http://twitter.com/ColeVsCode">@colevscode</a>
disqusid: front-end-frameworks
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

Lately we've been hearing a lot of buzz about Angular.js, and have been itching to try it on a larger project. In the past we've primarily used Backbone.js. Our entire site and most of our templates are written in backbone. But we've also played around with angular, and have written a few angular tutorials that were quite popular. (Especially our post on [Using AngularJS with a Backend API](/entry/angular-tut2) that hit the #1 spot on google for "angular backend") With an overhaul of the Backlift website on the horizon, we figured it was time to give angular a more serious look.

Before laying down code, we wanted to find out more information. What other startups are using Angular? How big are those companies, and what backend stacks are they running? We also wanted to know what other frontend tools startups are using, like SASS or Coffeescript. To answer these questions we sent a poll out to hundreds of other startup founders and friends. (primarily YC companies) We received over 30 responses and here are the results:

## What's your frontend stack?
 
{{$ with '{"name": "fw", "file": "/data/frameworks.csv"}' as data }}
	{{$ include /partials/d3bargraphs.html }}
{{$ endwith }}

We were struck by the fact that two thirds of companies that responded were using some kind of frontend MVC framework. Backbone was the most popular, and was also favored by larger startups, as determined by the size of their development teams. The majority of smaller startups, on the other hand, were either using Angular, or no MVC framework. It seems from this relatively small sampling that Backbone remains the dominant MVC framework for now, but Angular definitely has a significant following. Perhaps Angular's double-binding and reduced boilerplate are compelling features for small teams with severely limited time.

We also asked what backend stack startups were using. We had a theory that companies using a frontend MVC framework would be more likely to use a lightweight server, one that merely provides a RESTful API layer over the database. (That's what we're dong here at Backlift) Here's what we discovered:

{{$ with '{"name": "be", "file": "/data/backends.csv"}' as data }}
	{{$ include /partials/d3bargraphs.html }}
{{$ endwith }}

... Basically we saw an even split. It doesn't appear from this data that the choice of frontend framework affected the choice to use an MVC backend stack.

Finally, we were curious which startups used frontend compiled languages like coffeescript, SASS and LESS. We also wanted to know whether the presence of designers on the team had an impact. Here's what we found:

{{$ include /partials/drawpara.html }}

{{$ with '{"name": "des", "data":["designers", "styles", "scripts"]}' as data }}
	{{$ include /partials/paraset.html }}
{{$ endwith }}

Most startups that responded were using some kind of intermediate CSS language. SASS was the clear winner, with more than half of the startups using it alone. The startups without designers made up the majority of LESS users, while startups with at least one designer were especially unlikely to use plain CSS. The use of plain old javascript was slightly higher than coffeescript. 

## Conclusions

Overall the majority of startups that responded to our survey were using some kind of frontend MVC framework, the most popular of which was Backbone.js. Also startups were more likely to use SASS or LESS than plain old CSS. Beyond that, it was difficult to draw sweeping conclusions. However, we may have missed something, so we're including the full dataset [here](/data/data.csv), along with a <a href="" id="kitchensink">kitchen-sink visualization of the data</a>. (Tufte, please forgive us)

{{$ with '{"name": "all", "style":"display: none; margin-bottom: 10px", "data":["framework", "devs", "designers", "backend", "scripts"]}' as data }}
	{{$ include /partials/paraset.html }}
{{$ endwith }}

<script>
$('#kitchensink').click(function(ev) {
	ev.preventDefault();
	$("#visall svg").show("slow");
});
</script>


We'd love to hear what you're using. Please fill out the survey below. If the results reveal a different trend, we'll be sure to post an update. Also feel free to tell us what frontend stack you're using by posting in the [Hacker News comments]().


<h2><a id="survey"></a>Tell us about your frontend stack!</h2>
<p>

{{$ if not {$ form.success } }}

	<form method="post" action="/backlift/data/frontstackresponses">
		{{$csrf}}
		<div class="row-fluid">
			<div class="span6">
				<label>Which frontend framework do you use at your company?</label>
				<select class="span12" name="framework">
					<option value="">-- choose one --</option>
					<option>Angular</option>
					<option>Backbone</option>
					<option>Ember</option>
					<option>Other</option>
					<option>None</option>
				</select>
			</div>
			<div class="span6">
				<label>Do you use coffeescript or javascript at your company?</label>
				<select class="span12" name="scripts">
					<option value="">-- choose one --</option>
					<option>Javascript</option>
					<option>Coffeescript</option>
					<option>Other</option>
				</select>
			</div>
		</div>
		<div class="row-fluid">
			<div class="span6">
				<label>What CSS language do you use? (SASS, LESS, etc...)</label>
				<input class="span12" type="text" name="styles" value="{$ form.data.styles }"/>
			</div>
			<div class="span6">
				<label>What is your backend stack? (rails, node, django etc...)</label>
				<input class="span12" type="text" name="backend" value="{$ form.data.backend }"/>
			</div>
		</div>
		<div class="row-fluid">
			<div class="span6">
				<label>How many full-time developers are in the company?</label>
				<input class="span12" type="text" name="devs" value="{$ form.data.devs }"/>
			</div>
			<div class="span6">
				<label>How many full-time designers are in the company?</label>
				<input class="span12" type="text" name="designers" value="{$ form.data.designers }"/>
			</div>
		</div>
		<div class="row-fluid">
			<div class="span6">
				<label>What's the name of the company? <br>(For verification. We won't share this.)</label>
				<input class="span12" type="text" name="company" value="{$ form.data.company }"/>
			</div>
			<div class="span6">
				<label>What's your email address? <br>(If we have questions. We won't share this.)</label>
				<input class="span12" type="text" name="email" value="{$ form.data.email }"/>
			</div>
		</div>
		{{$formerrors}}
		<p/>
		<p><button class="btn btn-primary">Submit and see the results!</button></p>
		<input type="hidden" name="_next" value="/entry/front-end-frameworks#survey"/>
		<input type="hidden" name="_retry" value="/entry/front-end-frameworks#survey"/>
	</form>

{{$ endif }}

{{$ if {$ form.success } }}
	{{$ formsuccess }}

	<label>Results:</label>

	{{$ with '{"name": "user", "url": "/backlift/data/frontstackresponses", "columns": {"framework": ["angular", "backbone", "ember", "other", "none"] }}' as data }}
		{{$ include /partials/d3minibargraphs.html }}
	{{$ endwith }}

{{$ endif }}

</p>

