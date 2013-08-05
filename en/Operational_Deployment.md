# Deloyment

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

	More information: [Supervisord](Supervisord.md).

- nohup

	nohup ./beepkg &