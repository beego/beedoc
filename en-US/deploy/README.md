---
root: true
name: Deployment
sort: 7
---

# Releasing and Deploying

### Development mode

The application created by `bee` is in development mode by default.

We can change the mode by:

	beego.RunMode = "prod"

Or change it in conf/app.conf:

	runmode = prod


In development mode:

- If you don't have a views folder, it will show this kind of error:

		2013/04/13 19:36:17 [W] [stat views: no such file or directory]

- Templates will load every time without cache.

- If server throws error, the response will look like:

![](./../images/dev.png)

### Releasing and Deploying

The Go application is a bytecode file after compiling. You just need to copy this file to the server and run it. But remember Beego might also include static files, configuration files and templates, so these three folders also need to be copied to server while deploying.

	$ mkdir /opt/app/beepkg
	$ cp beepkg /opt/app/beepkg
	$ cp -fr views /opt/app/beepkg
	$ cp -fr static /opt/app/beepkg
	$ cp -fr conf /opt/app/beepkg

Here is the folder structure in `/opt/app/beepkg`:

	.
	├── conf
	│   ├── app.conf
	├── static
	│   ├── css
	│   ├── img
	│   └── js
	└── views
	    └── index.tpl
	├── beepkg

Now we've copied our entire application to the server. Next step is deploy it.

There are two ways to run it:

- [Stand alone deploy](./beego.md)
- [Deploy with Supervisord ](./supervisor.md)
	
The application is exposed above, then usually we will have a nginx or apache to serve pages and perform load balancing on our application.

- [Deploy with Nginx](./nginx.md)
- [Deploy with Apache](./apache.md)
