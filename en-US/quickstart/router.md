---
name: Routing settings
sort: 2
---

# Project routing settings

So, we've created a Beego project and ran it in the previous section. But how does it work? Let's investigate from the main file (main.go):

	package main
	
	import (
	        _ "quickstart/routers"
	        "github.com/astaxie/beego"
	)
	
	func main() {
	        beego.Run()
	}

At first, the code imports the package `quickstart/routers`. Let's have a look there (routers/router.go):

	package routers

	import (
	        "quickstart/controllers"
	        "github.com/astaxie/beego"
	)

	func init() {
	        beego.Router("/", &controllers.MainController{})
	}

In total, there are two relevant lines here. The first one registers the router by calling `beego.Router` and the second one is `beego.Run`. What did these two lines do?

1. Actually beego.Router registered an address. The first argument is the request uri, which is `/` here. This means that the request doesn't have any uri. The second argument is the Controller that will handle the requests for this uri. We can also register a router like this:

		beego.Router("/user", &controllers.UserController{})
The user can then visit `/user` to invoke the logic in UserController. That's how easy routers are. For further information on router usage please see [beego router settings](../mvc/controller/router.md).

2. After `beego.Run` is executed, we can see that it's listening on the port. Behind the scenes, it did a lot of work:
  - Parsing the [configuration file](../mvc/controller/config.md)
    Beego will parse the configuration file `app.conf` in conf folder. There we can change the port, enable session management and change the application's name by setting the corresponding options.

  - Initializing the [user session](../mvc/controller/session.md)
    Beego will initialize the user session, depending on the setting in the configuration file.

  - Compiling the [views](view.md)
    Beego will compile the views in the views folder when it's starting so as to avoid compiling them multiple times, which is definitely more efficient.

  - Starting the [supervisor module](../advantage/monitor.md)
    Beego has a very cool module which is called supervisor module. We can see the QPS, cpu, memory, GC, goroutine and thread information by visiting port 8088.

  - Listening on the service port
    This last step gets Beego listening for the http requests on port 8080. It takes advantage of goroutines by calling `ListenAndServe`.

  - After everything is running, our server will serve incoming requests from port 8080 and supervising from port 8088.

We saw the whole process of running a Beego project and some other functions. Let's see how the controller works in the [next section](controller.md).
