{{$ meta }}
title: major backlift upgrades roll out today
created: 2013-04-10
public: no
author: <a href="http://twitter.com/ColeVsCode">@colevscode</a>
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

We just pushed a major update to Backlift. Here's what's new:

- **Dropbox updates are now faster.** We now begin syncing in less than a second after we've detected changes to your Dropbox.
- **The new admin interface.** If you create a new app, you'll see the admin interface included has many new features including user management, a data browser, and import/export.
- **The new sync bar.** New apps come with a status tab on the bottom of the page that indicates Backlift sync status and last update time.

{{$ break }}

The downside is that the old style admin has been removed. Existing projects may see a 404 error when attempting to access the admin. If you desire to use the new admin with an existing project, just copy the /admin folder out of a new project and drop it into the existing one.
