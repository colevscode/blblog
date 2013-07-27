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

Before making any decisions we wanted a bit more information. Is angular trusted on real applcations? Are other startups staking their futures on it? To answer these questions, and to get a sense of where silicon valley is headed, we sent a poll out to hundreds of other startup founders and friends. We asked them if they were using Angular or Backbone, and what other front end technologies they used. We received 32 responses and here are the results:


### What's your front-end stack?


{{$ each '[["designers",  "scripts", "styles", "backend"]]' as columns }}
	{{$ include /partials/paraset.html }}
{{$ endeach }}
