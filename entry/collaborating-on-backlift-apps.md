{{$ meta }}
title: collaborating on backlift apps
created: 2013-06-18
public: yes
author: <a href="http://twitter.com/laurihy">@laurihy</a>
disqusid: fa5c882c-0372-4d6b-b401-8ce21e10292e
{{$ endmeta }}

{{$ layout /partials/blogentry.html as content }}

One common use case for Dropbox folders is collaborating and sharing files with other people. Dropbox doesn't support sharing folders inside Apps-folder, but it doesn't mean that you're on your own when working with Backlift. While we're building a better solution for collaborating on Backlift apps, here's a simple workaround:

{{$ break }}

1. Create an app on Backlift (unless you already have one), let's call it *myapp*
2. Move the myapp-folder `/Dropbox/Apps/Backlift/myapp` to Dropbox root: `/Dropbox/myapp`
3. Create a *symbolic link* `/Dropbox/Apps/Backlift/myapp` that points to `/Dropbox/myapp/`

For following commands, **make sure path to Dropbox is correct and change myapp to your app's folder.**

#### On OS X and Linux:

Open Terminal:
  
	mv ~/Dropbox/Apps/Backlift/myapp ~/Dropbox
	ln -s ~/Dropbox/myapp ~/Dropbox/Apps/Backlift/myapp

#### On Windows:

Open Command Prompt:

	move ~/Dropbox/Apps/Backlift/myapp ~/Dropbox
	mklink ~/Dropbox ~/Dropbox/Apps/Backlift/myapp

[Here's another, more elaborate tutorial](http://www.howtogeek.com/howto/16226/complete-guide-to-symbolic-links-symlinks-on-windows-or-linux/)

*(Windows version actually hasn't been tested. The principle should be same as on OS X and Linux, but we'd love to hear if there are some problems)*

## Finally

Now you can share the folder from Dropbox root with whomever you want, and any changes they make there will be synchronized normally to Backlift.
