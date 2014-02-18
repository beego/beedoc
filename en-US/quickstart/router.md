---
name: Routing setting
sort: 2
---

# Project routing setting

We created a beego project and ran it in previous section. But how does it work? Let's investigate from the starting file.

	package main
	
	import (
	        "quickstart/controllers"
	        "github.com/astaxie/beego"
	)
	
	func main() {
	        beego.Router("/", &controllers.MainController{})
	        beego.Run()
	}
	
We can see two simple lines. The first one register the router by `beego.Router` and the second one is `beego.Run`. What did these two lines do.

1. Actually beego.Router registered a address. The first argument is the request uri and here is `/` which means the request doesn't have any uri. The second argument is the Controller who will handle the requests for this uri. We can also register router like this:

		beego.Router("/user", &controllers.UserController{})	
Then the user can visit /user to process the logic in UserController.  This is our simple routers. For more router usage please see [beego router setting](../mvc/controller/router.md).
	
2. After `beego.Run` exected, we just saw it's listening the port. It did lots of work indeed.
  - Parsing configuration file
	
    Beego will parse the configuration file `app.conf` in conf folder. Then we can change the port, enable the session and change the application's name by setting up the configuration file.

	- Enabling session
    It will enable or disable the session by the setting in in the configuration file

	- Compiling the views 
    Beego will compile the views in views folder when it's starting running so as to avoid compile multiple times. It is more efficient for processing the views.
	
  - Starting superviser module
    Beego has a very cool module which is called superviser module. We can see the QPS, cpu, memory, GC, goroutine, thread information by visiting port 8088.

  - Listening service port
    This last step will listening the 8080 port which is for the http requests. It takes the advantages of goroutine by calling `ListenAndServe` indeed.
	
  - After everything is running, our server will serve outcoming requests from port 8080 and supervising from port 8088.
	
We saw the whole process of running a beego project and some other functions. Let's start to see how does the Controller work.
