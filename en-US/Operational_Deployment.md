# Deployment

### Development mode

Beego uses development mode as default, you can use following code to change mode in your application:

	beego.RunMode = "prod"

Or use configuration file in `conf/app.conf`, and input following content:

	runmode = prod

No differences between two ways.

In development mode, you have following effects:

- If you don't have directory `views`, it prints following error prompt:

		2013/04/13 19:36:17 [W] [stat views: no such file or directory]

- It doesn't cache template and reload every time.
- If panic occurs in your server, it prints information like following screen shot:

![](https://raw.github.com/astaxie/beego/master/docs/en/images/dev.png)

### Deloyment

Go compiles program to binary file, you only need to copy this binary to your server and run it. Because Beego uses MVC model, so you may have folders for static files, configuration files and template files, so you have to copy those files as well. Here is a real example for deployment.

	$ mkdir /opt/app/beepkg
	$ cp beepkg /opt/app/beepkg
	$ cp -fr views /opt/app/beepkg
	$ cp -fr static /opt/app/beepkg
	$ cp -fr conf /opt/app/beepkg

Here is the directory structure pf `/opt/app/beepkg`.

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

Now you can run your application in server, here are two good ways to manage your applications, and I recommend the first one.

- Supervisord 

	More information: [Supervisord](/docs/Operational_Supervisord).

- nohup

	nohup ./beepkg &
