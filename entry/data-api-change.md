{{$ meta }}
title: backlift data API change
created: 2013-04-14
public: yes
author: <a href="http://twitter.com/ColeVsCode">@colevscode</a>
disqusid: 62488fa6-e453-4a9d-8746-291399c3c2db
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

This is a quick note that we're modifying the data persistence API. Until now we've had two separate API schemes. Most of our API endpoints started with `/backlift/` except for our data persistence API which started with `/backliftapp/`. Moving forward we will move the data persistence API from `/backliftapp/` to `/backlift/data/`. This will unify all our API endpoints under a single `/backlift/` namespace. Regardless, applications will continue to work using the old API for the time being. {{$ break }}

Originally, the thinking behind the `/backliftapp/` namespace was this: By creating collections you are defining a custom API for your app since each collection gets its own URL. So the data persistence API is in fact a unique API defined by your app, hence the `/backlifapp/` namespace. However, this turns out to be confusing in practice. It's easier to just keep everything under one namespace, and the `/data` path clarifies the purpose of those API endpoints.