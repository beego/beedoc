---
name: Routing
sort: 2
---

# Routing

When do we set the router? When we discuss MVC structure of beego, we learned there are three type of router in Beego. Let's see how to use them now.

## RESTful router

Let's talk about RESTful first. RESTful is a popular way in API development. Beego supports it implict. Executing Get method for GET request and Post method for POST request. The defaut router is RESTful.

## Fixed router

Fixed router is full matching router, such as:

	beego.Router("/", &controllers.MainController{})
	beego.Router("/admin", &admin.UserController{})
	beego.Router("/admin/index", &admin.ArticleController{})
	beego.Router("/admin/addpkg", &admin.AddController{})
	
The fixed routers above are most common routers. One fixed router, one controller. Then execute different method based on different request method. It's a typical RESTFul router.

## Regex router

In order to make the router settings easier, beego reference the router implementation in Sinatra. It supports many router types.

- beego.Router(“/api/:id([0-9]+)“, &controllers.RController{})

  Customized regex //matching /api/123 :id = 123

- beego.Router(“/news/:all”, &controllers.RController{})

  Full matching //matching /news/path/to/123.html :all = path/to/123.html

- beego.Router(“/user/:username([\w]+)“, &controllers.RController{})

  Regex string matching //matching /user/astaxie :username = astaxie

- beego.Router(“/download/.”, &controllers.RController{})

  *matching //matching /download/file/api.xml :path= file/api :ext=xml

- beego.Router(“/download/ceshi/*“, &controllers.RController{})

  *full matching //matching /download/ceshi/file/api.json :splat=file/api.json

- beego.Router(“/:id:int”, &controllers.RController{})

  int type matching //matching :id is int type. Beego implemented ([0-9]+) for you

- beego.Router(“/:hello:string”, &controllers.RController{})

  string type matching //matching :hello is string type. Beego implemented ([\w]+) for you

In controller, you can get the variables like this:

	this.Ctx.Input.Param(":id")
	this.Ctx.Input.Param(":username")
	this.Ctx.Input.Param(":splat")
	this.Ctx.Input.Param(":path")
	this.Ctx.Input.Param(":ext")

## Custom methods and RESTful rules

The examples above are using default method name (the request methods name is same as the controller method name, such as `GET` request execute `Get` method and `POST` request execute `Post` method). If you want to use different controller method name, you can do this:

	beego.Router("/",&IndexController{},"*:Index")

Use the third parameter which is the method you want to call in the controller. Here is some rules:

* * means any request method will all execute this method.
* Use httpmethod:funcname format
* For more format, use `;` as the separator
* Many method mapping the same funcname, use `,` as the separator

Below is some example of RESTFul design:

	beego.Router("/api/list",&RestController{},"*:ListFood")
	beego.Router("/api/create",&RestController{},"post:CreateFood")
	beego.Router("/api/update",&RestController{},"put:UpdateFood")
	beego.Router("/api/delete",&RestController{},"delete:DeleteFood")

Below is the example of multiple HTTP methods mapping to a same method:

	beego.Router("/api",&RestController{},"get,post:ApiFunc")

Below is different HTTP methods mapping to different methods. `;` as the separator:

	beego.Router("/simple",&SimpleController{},"get:GetFunc;post:PostFunc")

Below is the acceptable HTTP methods:

* *：including all methods below
* get ：GET request
* post ：POST request
* put ：PUT request
* delete ：DELETE request
* patch ：PATCH request
* options ：OPTIONS request
* head ：HEAD request 

If * and other HTTP methods are used together, HTTP method will be executed prior. For example:

	beego.Router("/simple",&SimpleController{},"*:AllFunc;post:PostFunc")

The `PostFunc` other than `AllFunc` will be execute for POST request.

The router of custom methods don't support RESTFul by default which means if you set router like `beego.Router("/api",&RestController{},"post:ApiFunc")` and the request method is `POST` then the `Post` method won't be executed by default.

## Auto matching

Firstly you need to register controller into auto router.

	beego.AutoRouter(&controllers.ObjectController{})

Then Beego will retrive all the methods in that controller by reflection and you can call the related methods by this:

	/object/login   will call Login method in ObjectController 
	/object/logout  will call Logot method in ObjectController

Except `/:controller/:method` will match to controller and method, all the rest of url path will be parsed as GET parameters and saved into `this.Ctx.Input.Param`:

	/object/blog/2013/09/12  will call Blog method in ObjectController with parameters map[0:2013 1:09 2:12]. 

URL will match by lowercase, so `object/LOGIN` will also map to `Login` method.

So for all the urls below will map to `simple` method in `controller`.

	/controller/simple
	/controller/simple.html
	/controller/simple.json
	/controller/simple.rss

Then you can get the extension name of the url by `this.Ctx.Input.Param(":ext")`.
