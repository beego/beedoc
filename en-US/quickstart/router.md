---
name: Routing setting
sort: 2
---

# Project routing setting

So, we've created a beego project and ran it in previous section. But how does it work? Let's investigate from the starting file (main.go).

	package main
	
	import (
	        "quickstart/controllers"
	        "github.com/astaxie/beego"
	)
	
	func main() {
	        beego.Router("/", &controllers.MainController{})
	        beego.Run()
	}
	
We can see two simple lines. The first one registers the router by calling `beego.Router` and the second one is `beego.Run`. What did these two lines do?

1. Actually beego.Router registered an address. The first argument is the request uri and here is `/` which means the request doesn't have any uri. The second argument is the Controller who will handle the requests for this uri. We can also register a router like this:

		beego.Router("/user", &controllers.UserController{})	
Then the user can visit /user to process the logic in UserController.  This is our simple routers. For further information on router usage please see [beego router setting](../mvc/controller/router.md).
	
2. After `beego.Run` executed, we just saw it's listening on the port. It did lots of work indeed.
  - Parsing configuration file
	
    Beego will parse the configuration file `app.conf` in conf folder. There we can change the port, enable the session and change the application's name by setting up the configuration file.

	- Enabling session
    Beego will enable or disable the session by the setting in in the configuration file.

	- Compiling the views 
    Beego will compile the views in views folder when it's starting so as to avoid compiling them multiple times, which is definitely more efficient.
	
  - Starting the supervisor module
    Beego has a very cool module which is called supervisor module. We can see the QPS, cpu, memory, GC, goroutine, thread information by visiting port 8088.

  - Listening on the service port
    This last step gets beego listening for the http requests on port 8080. It takes advantage of goroutines by calling `ListenAndServe` indeed.
	
  - After everything is running, our server will serve incoming requests from port 8080 and supervising from port 8088.
	
We saw the whole process of running a beego project and some other functions. Let's start to see how the Controller works.
