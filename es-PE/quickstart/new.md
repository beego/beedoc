---
name: Create new project
sort: 1
---

# Creating project

Most of beego projects are created with the `bee` command. Before starting anything, please make sure that you've already installed the `bee` tool and the `beego` package. If you don't have them yet, please read [Installing beego](../install) and [Installing bee tool](../install/bee.md) before you proceed.

When you are ready, we can get started now. Open your terminal then go to your `$GOPATH` directory and then type `bee new quickstart`:

	➜  src  bee new quickstart
	[INFO] Creating application...
	/gopath/src/quickstart/
	/gopath/src/quickstart/conf/
	/gopath/src/quickstart/controllers/
	/gopath/src/quickstart/models/
	/gopath/src/quickstart/static/
	/gopath/src/quickstart/static/js/
	/gopath/src/quickstart/static/css/
	/gopath/src/quickstart/static/img/
	/gopath/src/quickstart/views/
	/gopath/src/quickstart/conf/app.conf
	/gopath/src/quickstart/controllers/default.go
	/gopath/src/quickstart/views/index.tpl
	/gopath/src/quickstart/main.go
	13-11-26 10:34:10 [SUCC] New application successfully created!
	
The bee tool created a new beego project for you. Here is the structure:

	quickstart
	├── conf
	│   └── app.conf
	├── controllers
	│   └── default.go
	├── main.go
	├── models
	├── static
	│   ├── css
	│   ├── img
	│   └── js
	└── views
	    └── index.tpl	

We can tell this is a typical MVC application. `main.go` is the project's starting file.

## Running project

After creating the project, we can run our project now. Go to the path of the new created project and run `bee run` to run the project. It will also compile the project.

	➜  src  cd quickstart
	➜  quickstart  bee run
	13-11-26 10:43:14 [INFO] Uses 'quickstart' as 'appname'
	13-11-26 10:43:14 [INFO] Initializing watcher...
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart/controllers)
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart/models)
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart)
	13-11-26 10:43:14 [INFO] Start building...

We have a web application running on port `8080` (the default port of Beego) now. It's amazing, isn't it? Why can we do that without nginx or apache. Yes, you are right. Go has already implemented all the functions we need for network layer and beego encapsulated them so we don't need nginx or apache here. Let's see our application in browser now:

![](../images/beerun.png)

Are you excited? Isn't it so easy to create web applications? Let's dive into the project and see how does everything work.
