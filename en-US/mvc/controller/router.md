---
name: Routing
sort: 2
---

# Routing

When do we set the router? When we discuss MVC structure of beego, we learned there are three type of router in Beego. Let's see how to use them now.

## Basic router
from beego1.2 we support RESTFul function router。the basic router include the URI and closure function.

### GET router

```
beego.Get("/",func(ctx *context.Context){
     ctx.Output.Body([]byte("hello world"))
})
```
	
### POST router

```
beego.Post("/alice",func(ctx *context.Context){
     ctx.Output.Body([]byte("bob"))
})
```

### support all HTTP router

```
beego.Any("/foo",func(ctx *context.Context){
     ctx.Output.Body([]byte("bar"))
})
```

### all the function

* beego.Get(router, beego.FilterFunc)
* beego.Post(router, beego.FilterFunc)
* beego.Put(router, beego.FilterFunc)
* beego.Head(router, beego.FilterFunc)
* beego.Options(router, beego.FilterFunc)
* beego.Delete(router, beego.FilterFunc)
* beego.Any(router, beego.FilterFunc)

### Handler register
sometime we already use the `net/http` or other packages implemented our system. but we want to intergreted into beego API or web system. Now it's super easy to do that like this:

```
s := rpc.NewServer()
s.RegisterCodec(json.NewCodec(), "application/json")
s.RegisterService(new(HelloService), "")
beego.Handler("/rpc", s)
```

`beego.Handler(router, http.Handler)`,the first param represent URI,the second param represent `http.Handler`,when you register this, then all the request `/rpc` will call `http.Handler`.

in fact there's third param `isPrefix` ,the default value is `false`, if you set to `true`,then all the match will prefix matching, So the url `/rpc/user` will also call the register.

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

- beego.Router("/api/:id", &controllers.RController{})

  defaut   //matching /api/123    :id = 123  can matching /api/
	
- beego.Router("/api/:id!", &controllers.RController{})

  defaut //matching /api/123    :id = 123  can't matching /api/
	
- beego.Router("/api/:id([0-9]+)", &controllers.RController{})

  Customized regex //matching /api/123 :id = 123

- beego.Router("/news/:all", &controllers.RController{})

  Full matching //matching /news/path/to/123.html :all = path/to/123.html

- beego.Router("/user/:username([\w]+)", &controllers.RController{})

  Regex string matching //matching /user/astaxie :username = astaxie

- beego.Router("/download/\*.\*", &controllers.RController{})

  *matching //matching /download/file/api.xml :path= file/api :ext=xml

- beego.Router("/download/ceshi/*", &controllers.RController{})

  *full matching //matching /download/ceshi/file/api.json :splat=file/api.json

- beego.Router("/:id:int", &controllers.RController{})

  int type matching //matching :id is int type. Beego implemented ([0-9]+) for you

- beego.Router("/:hello:string", &controllers.RController{})

  string type matching //matching :hello is string type. Beego implemented ([\w]+) for you

- beego.Router("/cms_:id([0-9]+).html", &controllers.CmsController{})

  has prefix regex. :id is the regex. //matching cms_123.html :id = 123

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

## namespace

```
//init namespace
ns := beego.NewNamespace("/v1").
   Cond(func (ctx *context.Context) bool{
	   if ctx.Input.Domain() == "api.beego.me" {
		 return true
	   }
	   return false
   }).
   Filter("before", auth).   
   Get("/notallowed", func(ctx *context.Context) {
   	ctx.Output.Body([]byte("notAllowed"))
   }).
   Router("/version", &AdminController{}, "get:ShowAPIVersion").
   Router("/changepassword", &UserController{}).
   Namespace(
   	  beego.NewNamespace("/shop").
       	Filter("before", sentry).
       	Get("/:id", func(ctx *context.Context) {
       		ctx.Output.Body([]byte("notAllowed"))
   		}))
//register namespace
beego.AddNamespace(ns)
beego.Run()
```
the code showed support the URL:

* GET /v1/notallowed
* GET /v1/version
* GET /v1/changepassword
* POST /v1/changepassword
* GET /v1/shop/123

namespace support filter, condition,and nest namespace

namespace API:

- NewNamespace(prefix string)

	get namespace object,the follow API is namespace's method

- Cond(cond namespaceCond)  

	if the namespaceCond return true will run this namespace,unwise willn't run
	
- Filter(action string, filter FilterFunc)

	action represent which position to run ,`before` and `after` is two validate value
	
- Router(rootpath string, c ControllerInterface, mappingMethods ...string)

	
- AutoRouter(c ControllerInterface)
- AutoPrefix(prefix string, c ControllerInterface)
- Get(rootpath string, f FilterFunc)
- Post(rootpath string, f FilterFunc)
- Delete(rootpath string, f FilterFunc)
- Put(rootpath string, f FilterFunc)
- Head(rootpath string, f FilterFunc)
- Options(rootpath string, f FilterFunc)
- Patch(rootpath string, f FilterFunc)
- Any(rootpath string, f FilterFunc)
- Handler(rootpath string, h http.Handler)

	these function is the same as mentioned earlier

- Namespace(ns *Namespace)

	nest namespace	
	
	```
	ns := beego.NewNamespace("/v1").
	    Namespace(
	        beego.NewNamespace("/shop").
	            Get("/:id", func(ctx *context.Context) {
	                ctx.Output.Body([]byte("shopinfo"))
	        }),
	        beego.NewNamespace("/order").
	            Get("/:id", func(ctx *context.Context) {
	                ctx.Output.Body([]byte("orderinfo"))
	        }),
	        beego.NewNamespace("/crm").
	            Get("/:id", func(ctx *context.Context) {
	                ctx.Output.Body([]byte("crminfo"))
	        }),
	    )
	```	