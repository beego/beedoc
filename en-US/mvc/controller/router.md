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
	/object/logout  will call Logout method in ObjectController

Except `/:controller/:method` will match to controller and method, all the rest of url path will be parsed as GET parameters and saved into `this.Ctx.Input.Param`:

	/object/blog/2013/09/12  will call Blog method in ObjectController with parameters map[0:2013 1:09 2:12]. 

URL will match by lowercase, so `object/LOGIN` will also map to `Login` method.

So for all the urls below will map to `simple` method in `controller`.

	/controller/simple
	/controller/simple.html
	/controller/simple.json
	/controller/simple.rss

Then you can get the extension name of the url by `this.Ctx.Input.Param(":ext")`.

## Annotations
Since beego1.3  supports the annotations routing, users do not need to register in the router, you  can use Include function, let's look at the example below:

```
// CMS API
type CMSController struct {
	beego.Controller
}

func (c *CMSController) URLMapping() {
	c.Mapping("StaticBlock", c.StaticBlock)
	c.Mapping("AllBlock", c.AllBlock)
}


// @router /staticblock/:key [get]
func (this *CMSController) StaticBlock() {

}

// @router /all/:key [get]
func (this *CMSController) AllBlock() {

}
```

in the `router.go` we can register like this：

	beego.Include(&CMSController{})


Beego automated source code analysis, note will only be generated in dev mode.

So the above routing will support the following routing:

* GET /staticblock/:key
* GET /all/:key

Actually effect and their registration by the Router function is the same:

	beego.Router("/staticblock/:key", &CMSController{}, "get:StaticBlock")
	beego.Router("/all/:key", &CMSController{}, "get:AllBlock")
	
pay attention to the new version `URLMapping` function, this is the new increase of function, users without registration, then it can be run through the reflection to the corresponding function, if  registration will be executed by interface functions, the performance will improve a lot.

## namespace

```
//init namespace
ns := 
beego.NewNamespace("/v1",
	beego.NSCond(func(ctx *context.Context) bool {
		if ctx.Input.Domain() == "api.beego.me" {
			return true
		}
		return false
	}),
	beego.NSBefore(auth),
	beego.NSGet("/notallowed", func(ctx *context.Context) {
		ctx.Output.Body([]byte("notAllowed"))
	}),
	beego.NSRouter("/version", &AdminController{}, "get:ShowAPIVersion"),
	beego.NSRouter("/changepassword", &UserController{}),
	beego.NSNamespace("/shop",
		beego.NSBefore(sentry),
		beego.NSGet("/:id", func(ctx *context.Context) {
			ctx.Output.Body([]byte("notAllowed"))
		}),
	),
	beego.NSNamespace("/cms",
		beego.NSInclude(
			&controllers.MainController{},
			&controllers.CMSController{},
			&controllers.BlockController{},
		),
	),
)
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
* GET /v1/cms/ MainController、CMSController、BlockController annotation Router

namespace support filter, condition,and nest namespace

namespace API:

- NewNamespace(prefix string, funcs ...interface{})

	get namespace object. it is strongly recommended to use NS registered at the beginning of the corresponding function, because it is easier to through gofmt tools to see more clearly the routing level relationship
	
- NSCond(cond namespaceCond)

	if the namespaceCond return true will run this namespace,unwise willn't run
	
- NSBefore(filiterList ...FilterFunc)
- NSAfter(filiterList ...FilterFunc)

	corresponding beforeRouter and FinishRouter two filters, can be registered at the same time more filters
	
- NSInclude(cList ...ControllerInterface)
- NSRouter(rootpath string, c ControllerInterface, mappingMethods ...string)
- NSGet(rootpath string, f FilterFunc)
- NSPost(rootpath string, f FilterFunc)
- NSDelete(rootpath string, f FilterFunc)
- NSPut(rootpath string, f FilterFunc)
- NSHead(rootpath string, f FilterFunc)
- NSOptions(rootpath string, f FilterFunc)
- NSPatch(rootpath string, f FilterFunc)
- NSAny(rootpath string, f FilterFunc)
- NSHandler(rootpath string, h http.Handler)
- NSAutoRouter(c ControllerInterface)
- NSAutoPrefix(prefix string, c ControllerInterface)
	
	these function is the same as mentioned earlier
	
- NSNamespace(prefix string, params ...innnerNamespace)

	nest namespace
	
	```
	ns := 
  	beego.NewNamespace("/v1",
		beego.NSNamespace("/shop",
			beego.NSGet("/:id", func(ctx *context.Context) {
				ctx.Output.Body([]byte("shopinfo"))
			}),
		),
		beego.NSNamespace("/order",
			beego.NSGet("/:id", func(ctx *context.Context) {
				ctx.Output.Body([]byte("orderinfo"))
			}),
		),
		beego.NSNamespace("/crm",
			beego.NSGet("/:id", func(ctx *context.Context) {
				ctx.Output.Body([]byte("crminfo"))
			}),
		),
	)
	```	
	

the follow API is namespace's method：Directly use is not recommended, of course, the effect and the NS prefix function is the same, just the way is more elegant, the code easier to read

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
