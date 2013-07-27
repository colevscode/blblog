{{$ meta }}
title: single page apps crawled by google
created: 2013-04-18
public: no
author: <a href="http://twitter.com/ColeVsCode">@colevscode</a>
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

Here's one datapoint on the ongoing debate about whether or not "single page apps" (or Javascript rendered websites) are crawled by Google: this website.

[Blog.backlift.com](http://blog.backlift.com) is a single page app written entirely in javascript and it's being crawled by Google. Here are the details: {{$ break }} The blog is written in backbone.js and the html is rendered via several handlebars templates (a top level layout and embedded subviews). We don't do anything fancy except to bootstrap data using the backlift {{$ prefetch }} tag. We also serve our site with the help of CloudFlare's basic CDN plan.

Here's a screenshot of our recent blog post on [Angular.js Views and Controllers](https://blog-colevscode.backliftapp.com/entry/angular-tut1):

<img src="/images/google-blog.png">


In other words, we've managed to build a Single Page App that loads quickly, is SEO friendly and doesn't require any server-side magic. (No phantom.JS rendering for SEO or crazy servier-side template processing). 

Does our site render as quickly as server rendered HTML? Probably not. Is it good enough? We'd love to know what you think-- shoot us an email at feedback at backlift.com.