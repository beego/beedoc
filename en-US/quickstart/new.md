---
name: Create a new project
sort: 1
---

# Creating a new project

Most Beego projects are created with the [`bee` command](../install/bee.md). Before starting anything, please make sure that you've already installed the `bee` tool and the `beego` package. If you don't have them yet, please read [Installing beego](../install) and [Installing bee tool](../install/bee.md) before you proceed.

When you are ready, we can get started now. Open your terminal, then go to your `$GOPATH` directory and then type `bee new quickstart`:

	➜  src  bee new quickstart
	[INFO] Creating application...
	/gopath/src/quickstart/
	/gopath/src/quickstart/conf/
	/gopath/src/quickstart/controllers/
	/gopath/src/quickstart/models/
	/gopath/src/quickstart/routers/
	/gopath/src/quickstart/tests/
	/gopath/src/quickstart/static/
	/gopath/src/quickstart/static/js/
	/gopath/src/quickstart/static/css/
	/gopath/src/quickstart/static/img/
	/gopath/src/quickstart/views/
	/gopath/src/quickstart/conf/app.conf
	/gopath/src/quickstart/controllers/default.go
	/gopath/src/quickstart/views/index.tpl
	/gopath/src/quickstart/routers/router.go
	/gopath/src/quickstart/tests/default_test.go
	2015/05/02 11:55:28 [SUCC] New application successfully created!

The bee tool created a new Beego project for you. Here is the structure:

	quickstart
	├── conf
	│   └── app.conf
	├── controllers
	│   └── default.go
	├── main.go
	├── models
	├── routers
	│   └── router.go
	├── static
	│   ├── css
	│   ├── img
	│   └── js
	├── tests
	│   └── default_test.go
	└── views
	    └── index.tpl

We can tell this is a typical MVC application. `main.go` is the project's main file.

## Running project

After creating the project, we can run our project now. Go to the path of the newly created project and run `bee run` to run the project. It will also compile the project.

	➜  src  cd quickstart
	➜  quickstart  bee run
	2015/05/02 12:01:31 [INFO] Uses 'quickstart' as 'appname'
	2015/05/02 12:01:31 [INFO] Initializing watcher...
	2015/05/02 12:01:31 [TRAC] Directory(/gopath/src/quickstart/controllers)
	2015/05/02 12:01:31 [TRAC] Directory(/gopath/src/quickstart)
	2015/05/02 12:01:31 [TRAC] Directory(/gopath/src/quickstart/routers)
	2015/05/02 12:01:31 [TRAC] Directory(/gopath/src/quickstart/tests)
	2015/05/02 12:01:31 [INFO] Start building...
	2015/05/02 12:01:36 [SUCC] Build was successful
	2015/05/02 12:01:36 [INFO] Restarting quickstart ...
	2015/05/02 12:01:36 [INFO] ./quickstart is running...
	2015/05/02 12:01:38 [app.go:103] [I] http server Running on :8080

We have a web application running on port `8080` (the default port of Beego) now. It's amazing, isn't it? We can do that without nginx or apache. Yes, you are right. Go has already implemented all the functions we need for the network layer and Beego encapsulated them so we don't need nginx or apache here. Let's look at our application in the browser now:

![](../images/beerun.png)

Are you excited? Isn't it so easy to create a web application? Let's dive into the project and see how everything works in the [next section](router.md).
