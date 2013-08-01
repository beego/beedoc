Beego makes it easy to build web applications using the Model-View-Controller (MVC) pattern by relying on conventions that require a certain structure in your application. In return, it is very light on configuration and enables an extremely fast development cycle.

### MVC

Here is a quick summary:

- Models are the essential data objects that describe your application domain. Models also contain domain-specific logic for querying and updating the data.
- Views describe how data is presented and manipulated. In our case, this is the template that is used to present data and controls to the user.
- Controllers handle the request execution. They perform the user’s desired action, they decide which View to display, and they prepare and provide the necessary data to the View for rendering.

There are many excellent overviews of MVC structure online. In particular, the one provided by Play! Framework matches our model exactly.

### Server

Beego builds on top of the Go HTTP server, which creates a go-routine (lightweight thread) to process each incoming request. The implication is that your code is free to block, but it must handle concurrent request processing.

### Filters

**Filters** implement most request processing functionality provided by Beego. The filter chain is an array of functions, each one invoking the next, until the terminal filter stage invokes the action selected by the router.

### Controllers and Actions

Each HTTP request invokes an **action**, which handles the request and writes the response. Related **actions** are grouped into **controllers**. The **[Controller](http://gowalker.org/github.com/astaxie/beego#Controller)** type contains relevant fields and methods and acts as the context for each request.

As part of handling a HTTP request, Beego instantiates an instance of your **Controller**, and it sets all of these properties on the embedded `beego.Controller`. Beego does not share **Controller** instances between requests.

A **Controller** is any type that embeds `revel.Controller` (directly or indirectly).

	type AppController struct {
	  beego.Controller
	}

An **Action** is any method on a **Controller** that has to be exported.

For example:

	func (this *AppController) Get() {
		..
	}
