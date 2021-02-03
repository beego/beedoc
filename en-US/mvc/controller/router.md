---
name: Routing
sort: 2
---

# Routing

This chapter will cover the three types of routers incorporated into Beego.

## Basic router
Beego supports a RESTful function router. This basic router includes the URI and closure functions.

### GET router

```
web.Get("/",func(ctx *context.Context){
     ctx.Output.Body([]byte("hello world"))
})
```

### POST router

```
web.Post("/alice",func(ctx *context.Context){
     ctx.Output.Body([]byte("bob"))
})
```

### support all HTTP routers

```
web.Any("/foo",func(ctx *context.Context){
     ctx.Output.Body([]byte("bar"))
})
```

### all the functions

* web.Get(router, web.FilterFunc)
* web.Post(router, web.FilterFunc)
* web.Put(router, web.FilterFunc)
* web.Head(router, web.FilterFunc)
* web.Options(router, web.FilterFunc)
* web.Delete(router, web.FilterFunc)
* web.Any(router, web.FilterFunc)

### Handler register
In cases where packages such as `net/http` are already implemented in a system they can be integrated into the web API or web system by following this procedure:

```
s := rpc.NewServer()
s.RegisterCodec(json.NewCodec(), "application/json")
s.RegisterService(new(HelloService), "")
web.Handler("/rpc", s)
```

`beego.Handler(router, http.Handler)` the first parameter represents the URI, and the second parameter represents `http.Handler`. When this is registered all requests to `/rpc` will call `http.Handler`.

There is also a third parameter, `isPrefix`.  If this parameter is set to `true` all the matches will comply with prefix matching, meaning that the url `/rpc/user` will also call the register. By default this value is `false`.

## RESTful router

RESTful is a popular approach to API development that Beego supports implicitly, executing `Get` method for GET request and `Post` method for POST request. The default router is RESTful.

## Fixed router

A fixed router is a full matching router, such as:

	web.Router("/", &controllers.MainController{})
	web.Router("/admin", &admin.UserController{})
	web.Router("/admin/index", &admin.ArticleController{})
	web.Router("/admin/addpkg", &admin.AddController{})

The fixed routers above are typical RESTful routers in their most common configuration, with one fixed router and one controller. This results in the execution of a different method based on each request method.

## Regex router

To simplify router configuration, Beego uses the router implementation approach found in Sinatra to support many router types.

- web.Router("/api/?:id", &controllers.RController{})

  *default matching* /api/123    :id = 123  *can match* /api/

- web.Router("/api/:id", &controllers.RController{})

  *default matching* /api/123    :id = 123  *can't match* /api/

- web.Router("/api/:id([0-9]+)", &controllers.RController{})

  *Customized regex matching* /api/123 :id = 123

- web.Router("/user/:username([\w]+)", &controllers.RController{})

  *Regex string matching* /user/astaxie :username = astaxie

- web.Router("/download/\*.\*", &controllers.RController{})

  *matching* /download/file/api.xml :path= file/api :ext=xml

- web.Router("/download/ceshi/*", &controllers.RController{})

  *full matching* /download/ceshi/file/api.json :splat=file/api.json

- web.Router("/:id:int", &controllers.RController{})

  *int type matching* :id is int type. web implements ([0-9]+) for you

- web.Router("/:hello:string", &controllers.RController{})

  *string type matching* :hello is string type. web implements ([\w]+) for you

- beego.Router("/cms_:id([0-9]+).html", &controllers.CmsController{})

  *has prefix regex* :id is the regex. *matching* cms_123.html :id = 123

The variables can be accessed in the controller like this:

	this.Ctx.Input.Param(":id")
	this.Ctx.Input.Param(":username")
	this.Ctx.Input.Param(":splat")
	this.Ctx.Input.Param(":path")
	this.Ctx.Input.Param(":ext")

## Custom methods and RESTful rules

The examples above use default method names, where the request method name is same as the controller method name.  For example as `GET` request executes `Get` method and `POST` request executes `Post` method.  Different controller method names can be set like this:

	web.Router("/",&IndexController{},"*:Index")

Use the third parameter which is the method you want to call in the controller. Here are some rules:

* * means any request method will execute this method.
* Use httpmethod:funcname format.
* Multiple formats can use `;` as the separator.
* Many HTTP methods mapping the same funcname, use `,` as the separator for HTTP methods.

Below are some examples of RESTful design:

	web.Router("/api/list",&RestController{},"*:ListFood")
	web.Router("/api/create",&RestController{},"post:CreateFood")
	web.Router("/api/update",&RestController{},"put:UpdateFood")
	web.Router("/api/delete",&RestController{},"delete:DeleteFood")

Below is an example of multiple HTTP methods mapping to the same controller method:

	web.Router("/api",&RestController{},"get,post:ApiFunc")

Below is an example of different HTTP methods mapping to different controller methods. `;` as the separator:

	web.Router("/simple",&SimpleController{},"get:GetFunc;post:PostFunc")

Below are the acceptable HTTP methods:

* *：including all methods below
* get ：GET request
* post ：POST request
* put ：PUT request
* delete ：DELETE request
* patch ：PATCH request
* options ：OPTIONS request
* head ：HEAD request

If * and other HTTP methods are used together the HTTP method will be executed first. For example:

	web.Router("/simple",&SimpleController{},"*:AllFunc;post:PostFunc")

The `PostFunc` rather than the `AllFunc` will be executed for POST requests.

The router of custom methods does not support RESTful behaviour by default which means if you set the router like `web.Router("/api",&RestController{},"post:ApiFunc")` and the request method is `POST` then the `Post` method won't be executed by default.

## Auto matching

To use auto matching the controller must be registered as an auto-router.

	web.AutoRouter(&controllers.ObjectController{})

Beego will retrieve all the methods in that controller by reflection. The related methods can be called like this:

	/object/login   will call Login method of ObjectController
	/object/logout  will call Logout method of ObjectController

Except `/:controller/:method` will match to controller and method. The remainder of the url path will be parsed as GET parameters and saved into `this.Ctx.Input.Param`:

	/object/blog/2013/09/12  will call Blog method of ObjectController with parameters `map[0:2013 1:09 2:12]`.

URL will match by lowercase conversion, so `object/LOGIN` will also map to `Login` method.

All the urls below will map to the `simple` method of `controller`.

	/controller/simple
	/controller/simple.html
	/controller/simple.json
	/controller/simple.xml

The extension name of the url can be reached by accessing `this.Ctx.Input.Param(":ext")`.

## Annotations
Not all routers need to be registered inside `router.go`. Only the controller needs to be registered using `Include`. For example:

```
// CMS API
type CMSController struct {
    web.Controller
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

The routers can then be registered in `router.go`

    web.Include(&CMSController{})

Beego will parse the source code automatically when under dev mode.

The following routers will be supported:

* GET /staticblock/:key
* GET /all/:key

This is exactly same as registering by Router functions:

    web.Router("/staticblock/:key", &CMSController{}, "get:StaticBlock")
    web.Router("/all/:key", &CMSController{}, "get:AllBlock")

If you do not use `URLMapping` Beego will find the function by reflection, otherwise Beego will find the function with the must faster `interface`.

## Automatic Parameter Handling
Beego supports automatic injection of http request parameters as method arguments, and method return values as http responses. 
For example, defining the following controller method:

```
// @router /tasks/:id
func (c *TaskController) MyMethod(id int, field string) (map[string]interface{}, error) {
	if u, err := getObjectField(id, field); err == nil {
		return u, nil
	} else {
		return nil, context.NotFound
	}
}
```

will automatically route the http parameters id and field (i.e. `/tasks/5?field=name` ) to the correct method parameters, and will render the method return value as JSON. If the method returns an error it will be rendered as an http status code.
If the parameter does not exist in the http request it will be passed to the method as the zero value for that parameter, unless that parameter is marked as 'required' using annotations.  This will return an error without calling the method.
For more information, see [Parameters](params.md)

## Method Expression Router
The method expression router is to register routers by providing a controller method expresion. If the receiver of 
the controller method is a non-pointer type, then you can pass method expression as `pkg.controller.method`. If the receiver
of method is a pointer, then you need to pass method expression as `(*pkg.controller).method`. However, if you register router in
the same package as controller, then you don't need to provide `pkg`.

````golang
// Here are some examples:

type BaseController struct {
	web.Controller
}

func (b BaseController) Ping() {
	b.Data["json"] = "pong"
	b.ServeJSON()
}

func (b *BaseController) PingPointer() {
	b.Data["json"] = "pong_pointer"
	b.ServeJSON()
}

func main() {
	web.RouterGet("/ping", BaseController.Ping)
	web.RouterGet("/ping_pointer", (*BaseController).PingPointer)
	web.Run()
}
````

There are many other Method Expression Routers：

* web.RouterGet(router, pkg.controller.method)
* web.RouterPost(router, pkg.controller.method)
* web.RouterPut(router, pkg.controller.method)
* web.RouterPatch(router, pkg.controller.method)
* web.RouterHead(router, pkg.controller.method)
* web.RouterOptions(router, pkg.controller.method)
* web.RouterDelete(router, pkg.controller.method)
* web.RouterAny(router, pkg.controller.method)


It also provides namespace functions：

* web.NSRouterGet
* web.NSRouterPost
* ......

## namespace

```
//init namespace
ns :=
web.NewNamespace("/v1",
    web.NSCond(func(ctx *context.Context) bool {
        if ctx.Input.Domain() == "api.web.me" {
            return true
        }
        return false
    }),
    web.NSBefore(auth),
    web.NSGet("/notallowed", func(ctx *context.Context) {
        ctx.Output.Body([]byte("notAllowed"))
    }),
    web.NSRouter("/version", &AdminController{}, "get:ShowAPIVersion"),
    web.NSRouter("/changepassword", &UserController{}),
    web.NSNamespace("/shop",
        web.NSBefore(sentry),
        web.NSGet("/:id", func(ctx *context.Context) {
            ctx.Output.Body([]byte("notAllowed"))
        }),
    ),
    web.NSNamespace("/cms",
        web.NSInclude(
            &controllers.MainController{},
            &controllers.CMSController{},
            &controllers.BlockController{},
        ),
    ),
)

//register namespace
web.AddNamespace(ns)
```
the code set out above supports the URL:

* GET /v1/changepassword
* POST /v1/changepassword
* GET /v1/shop/123
* GET /v1/cms/ maps to annotation routers in MainController, CMSController, BlockController

namespace supports filter, condition and nested namespace

namespace API:

- NewNamespace(prefix string,...interface{})

    Create a namespace object. The namespace object's methods are listed below.
    For compatibility with the gofmt tool is is recommend that these method names begin with `NS`.

- NSCond(cond namespaceCond)

    if the namespaceCond returns true this namespace will be run.

- NSBefore(filterList ...FilterFunc)
- NSAfter(filterList ...FilterFunc)

    For `BeforeRouter` and `FinishRouter` filters. Multiple filters can be registered.

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

    These are methods to set up routers equivilant to the basic routers.

- NSNamespace(prefix string, params ...innnerNamespace)

    Nested namespaces

    ```
    ns :=
    web.NewNamespace("/v1",
        web.NSNamespace("/shop",
            web.NSGet("/:id", func(ctx *context.Context) {
                ctx.Output.Body([]byte("shopinfo"))
            }),
        ),
        web.NSNamespace("/order",
            web.NSGet("/:id", func(ctx *context.Context) {
                ctx.Output.Body([]byte("orderinfo"))
            }),
        ),
        web.NSNamespace("/crm",
            web.NSGet("/:id", func(ctx *context.Context) {
                ctx.Output.Body([]byte("crminfo"))
            }),
        ),
    )
    ```

The methods below are for the`*Namespace` object and are not recommended. They have the same functionality as methods with `NS`, but are less elegant and harder to read.

- Cond(cond namespaceCond)

	if the namespaceCond return true will run this namespace.

- Filter(action string, filter FilterFunc)

	action represents which position to run ,`before` and `after` is two validate value

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

	these functions are the same as mentioned earlier

- Namespace(ns ...*Namespace)


More functions can be nested:

```go

//APIS
ns :=
	web.NewNamespace("/api",
		//It should verify the encrypted request in the production using
		web.NSCond(func(ctx *context.Context) bool {
			if ua := ctx.Input.Request.UserAgent(); ua != "" {
				return true
			}
			return false
		}),
		web.NSNamespace("/ios",
			//CRUD Create, Read, Update and Delete
			web.NSNamespace("/create",
				// /api/ios/create/node/
				web.NSRouter("/node", &apis.CreateNodeHandler{}),
				// /api/ios/create/topic/
				web.NSRouter("/topic", &apis.CreateTopicHandler{}),
			),
			web.NSNamespace("/read",
				web.NSRouter("/node", &apis.ReadNodeHandler{}),
				web.NSRouter("/topic", &apis.ReadTopicHandler{}),
			),
			web.NSNamespace("/update",
				web.NSRouter("/node", &apis.UpdateNodeHandler{}),
				web.NSRouter("/topic", &apis.UpdateTopicHandler{}),
			),
			web.NSNamespace("/delete",
				web.NSRouter("/node", &apis.DeleteNodeHandler{}),
				web.NSRouter("/topic", &apis.DeleteTopicHandler{}),
			)
		),
	)

web.AddNamespace(ns)
```
