# Routing

## Default RESTful rules

The main function of router is to connect request URL and handler. Beego wrapped `Controller`, so it connects request URL and `ControllerInterface`. The `ControllerInterface` has following methods:

```go
type ControllerInterface interface {
	Init(ct *Context, cn string)
	Prepare()
	Get()
	Post()
	Delete()
	Put()
	Head()
	Patch()
	Options()
	Finish()
	Render() error
}
```

`beego.Controller` implements all of the above methods, so you just use this struct as anonymous field in your controller struct. Of course you can overload methods to implement more specific logic.

Users can use the following ways to register route rules:

```go
beego.Router("/", &controllers.MainController{})
beego.Router("/admin", &admin.UserController{})
beego.Router("/admin/index", &admin.ArticleController{})
beego.Router("/admin/addpkg", &admin.AddController{})
```

For more convenient routing rule configuration, Beego borrows ideas from sinatra, supporting more kinds of route rules as follows:

- Customized regular expression matching:

		beego.Router("/api/:id([0-9]+)", &controllers.RController{})

  Matches `/api/123`, `:id= 123`

- Match rest of all:

		beego.Router("/news/:all", &controllers.RController{})

  Matches `/news/path/to/123.html`,  `:all= path/to/123.html`

- Regular expression:

		beego.Router(`/user/:username([\w]+)`, &controllers.RController{})

  Matches `/user/astaxie`,  `:username = astaxie`

- Wildcard character:

		beego.Router("/download/`*`.`*`", &controllers.RController{})

  Matches `/download/file/api.xml`,  `:path= file/api, :ext=xml`

- Wildcard character match rest of all:

		beego.Router("/download/ceshi/`*`", &controllers.RController{})

  Matches `/download/ceshi/file/api.json`, `:splat=file/api.json`

- Match type int:

		beego.Router("/:id:int", &controllers.RController{})

  `:id` is int type, Beego uses regular expression ([0-9]+) automatically to match int type parameters.

- Match type string:

		beego.Router("/:hi:string", &controllers.RController{})

  `:hi` is string type, Beego uses regular expression `([\w]+)` automatically to match string type parameters.


You can get the parameters from Controller:

```go
this.Ctx.Input.Params(":id")
this.Ctx.Input.Params(":username")
this.Ctx.Input.Params(":splat")
this.Ctx.Input.Params(":path")
this.Ctx.Input.Params(":ext")
```

## Customized methods and RESTful rules

We listed default method name above(methods name are the same as HTTP methods name, like Get for GET requests, Post for POST requests). You can customize your methods as follows:

	beego.Router("/", &IndexController{}, "*:Index")

The 3rd argument indicates the corresponding HTTP method for your method and router rules.

- * means execute this method for all coming requests.
- Use format `<HTTP method>:<function name>` to define routers.
- Use `;` to split multiple pairs.
- If multiple HTTP method are routing to one method, use `,` to split them.

Here is a RESTful style design:

```go
beego.Router("/api/list", &RestController{}, "*:ListFood")
beego.Router("/api/create", &RestController{}, "post:CreateFood")
beego.Router("/api/update", &RestController{}, "put:UpdateFood")
beego.Router("/api/delete", &RestController{}, "delete:DeleteFood")
```

An example of multiple HTTP method route to one method:

	beego.Router("/api", &RestController{}, "get, post:ApiFunc")

Multiple pairs example:

	beego.Router("/simple", &SimpleController{}, "get:GetFunc;post:PostFunc")

Available HTTP methods:

- *: For all coming requests.
- get: GET request.
- post: POST request.
- put: PUT request.
- delete: DELETE request.
- patch: PATCH request.
- options: OPTIONS request.
- head: HEAD request.

If you defined * and corresponding HTTP method, beego chooses HTTP method as prior execution. For example:

	beego.Router("/simple", &SimpleController{}, "*:AllFunc;post:PostFunc")

In this example, PostFunc will be executed instead of AllFunc.

## Automated routing

You should register your controllers for auto-routing:

	beego.AutoRouter(&controllers.ObjectController{})

Then beego reflects all methods in `ObjectController` and register corresponding routers:

	/object/login   Call Login method of ObjectController.
	/object/logout  Call Logout method of ObjectController.

In addition to matching of two prefixes:`/:controller/:method`, Beego automatically resolves the remaining URL path segments as parameters, and saves them into `this.Ctx.Param`:

	/object/blog/2013/09/12  Call Blog method of ObjectController, and has the argument map[0:2013 1:09 2:12].

The URL will be converted lower case before matching; therefore, `/object/LOGIN` routes to Login method as well.

What's more, beego is able to identify URLs like following examples, and gives away requests to controller's simple method, in this case:

	/controller/simple
	/controller/simple.html
	/controller/simple.json
	/controller/simple.rss

Then use `this.Ctx.Input.Params[":ext"]` to get extension.
