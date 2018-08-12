---
name: Routing settings
sort: 2
---

# Project routing settings

The previous section covered creating and running a Beego project.  This section will investigate the operation of the main file (main.go):

	package main
	
	import (
	        _ "quickstart/routers"
	        "github.com/astaxie/beego"
	)
	
	func main() {
	        beego.Run()
	}

This code imports the package `quickstart/routers`. This file contains the following (routers/router.go):

	package routers

	import (
	        "quickstart/controllers"
	        "github.com/astaxie/beego"
	)

	func init() {
	        beego.Router("/", &controllers.MainController{})
	}

There are two relevant lines here; `beego.Router` and `beego.Run`.

1.  `beego.Router` is used to register a router address. This command accepts two arguments. The first argument specifes the request uri, which is `/` here to indicate that no uri is requested.  The second argument is used to indicate the Controller that will handle requests for this uri. 

Alternately, a router can be registered in this format:

		beego.Router("/user", &controllers.UserController{})
The user can visit `/user` to invoke the logic in UserController. For further information on router usage please see [beego router settings](../mvc/controller/router.md).

2. `beego.Run` will actively listen on the specified port when executed. The following tasks are performed behind the scenes upon execution:
  - Parse the [configuration file](../mvc/controller/config.md)
    Beego will parse the configuration file `app.conf` in conf folder to change the port, enable session management and set the application's name.

  - Initialize the [user session](../mvc/controller/session.md)
    Beego will initialize the user session, based on the setting in the configuration file.

  - Compile the [views](view.md)
    Beego will compile the views in the views folder.  This is done on startup to avoid compiling multiple times and improve efficiency.

  - Starting the [supervisor module](../advantage/monitor.md)
    By visiting port `8088` the user can access information about QPS, cpu, memory, GC, goroutine and thread information.

  - Listen on the service port
    Beego will listen http requests on port `8080`. It takes advantage of goroutines by calling `ListenAndServe`.

  - When the application is running our server will serve incoming requests from port `8080` and supervising from port `8088`.

The next section will cover the operation of the controller [next section](controller.md).
