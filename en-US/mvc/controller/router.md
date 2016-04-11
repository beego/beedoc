---
name: Routing
sort: 2
---

# Routing

When do we set the router? When we discussed the MVC structure of Beego, we learnt that there are three types of router in Beego. Let's see how to use them now.

## Basic router
From beego version 1.2 we began supporting a RESTful function router. The basic router includes the URI and closure function.

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

### all the functions

* beego.Get(router, beego.FilterFunc)
* beego.Post(router, beego.FilterFunc)
* beego.Put(router, beego.FilterFunc)
* beego.Head(router, beego.FilterFunc)
* beego.Options(router, beego.FilterFunc)
* beego.Delete(router, beego.FilterFunc)
* beego.Any(router, beego.FilterFunc)

### Handler register
Sometimes it may be that we already use the `net/http` or other packages implemented in our system. But we may want to integrate it into the Beego API or web system. Now it's super easy to do that like this:

```
s := rpc.NewServer()
s.RegisterCodec(json.NewCodec(), "application/json")
s.RegisterService(new(HelloService), "")
beego.Handler("/rpc", s)
```

`beego.Handler(router, http.Handler)`,the first parameter represents the URI, the second parameter represents `http.Handler`. When you register this, then all the requests to `/rpc` will call `http.Handler`.

In fact there is a third parameter `isPrefix`, whose default value is `false`. If you set this parameter to `true`,then all the matches will comply with prefix matching, So the url `/rpc/user` will also call the register.

## RESTful router

Let's talk about RESTful first. RESTful is a popular approach in API development. Beego supports it implicitly. Executing `Get` method for GET request and `Post` method for POST request. The default router is RESTful.

## Fixed router

Fixed router is a full matching router, such as:

	beego.Router("/", &controllers.MainController{})
	beego.Router("/admin", &admin.UserController{})
	beego.Router("/admin/index", &admin.ArticleController{})
	beego.Router("/admin/addpkg", &admin.AddController{})

The fixed routers above are the most common routers. One fixed router, one controller. This results in the execution of a different method based on each request method. It's a typical RESTful router.

## Regex router

In order to make the router settings easier, Beego references the router implementation approach found in Sinatra. It supports many router types.

- beego.Router("/api/?:id", &controllers.RController{})

  *default matching* /api/123    :id = 123  *can matching* /api/

- beego.Router("/api/:id", &controllers.RController{})

  *default matching* /api/123    :id = 123  *can't matching* /api/

- beego.Router("/api/:id([0-9]+)", &controllers.RController{})

  *Customized regex matching* /api/123 :id = 123

- beego.Router("/user/:username([\w]+)", &controllers.RController{})

  *Regex string matching* /user/astaxie :username = astaxie

- beego.Router("/download/\*.\*", &controllers.RController{})

  *matching* /download/file/api.xml :path= file/api :ext=xml

- beego.Router("/download/ceshi/*", &controllers.RController{})

  *full matching* /download/ceshi/file/api.json :splat=file/api.json

- beego.Router("/:id:int", &controllers.RController{})

  *int type matching* :id is int type. Beego implements ([0-9]+) for you

- beego.Router("/:hello:string", &controllers.RController{})

  *string type matching* :hello is string type. Beego implements ([\w]+) for you

- beego.Router("/cms_:id([0-9]+).html", &controllers.CmsController{})

  *has prefix regex* :id is the regex. *matching* cms_123.html :id = 123

In controller, you can get the variables like this:

	this.Ctx.Input.Param(":id")
	this.Ctx.Input.Param(":username")
	this.Ctx.Input.Param(":splat")
	this.Ctx.Input.Param(":path")
	this.Ctx.Input.Param(":ext")

## Custom methods and RESTful rules

The examples above use default method names (the request method name is same as the controller method name, such as `GET` request executes `Get` method and `POST` request executes `Post` method). If you want to use different controller method names, you can do this:

	beego.Router("/",&IndexController{},"*:Index")

Use the third parameter which is the method you want to call in the controller. Here are some rules:

* * means any request method will execute this method.
* Use httpmethod:funcname format.
* Multiple formats can use `;` as the separator.
* Many HTTP methods mapping the same funcname, use `,` as the separator for HTTP methods.

Below are some examples of RESTful design:

	beego.Router("/api/list",&RestController{},"*:ListFood")
	beego.Router("/api/create",&RestController{},"post:CreateFood")
	beego.Router("/api/update",&RestController{},"put:UpdateFood")
	beego.Router("/api/delete",&RestController{},"delete:DeleteFood")

Below is an example of multiple HTTP methods mapping to the same controller method:

	beego.Router("/api",&RestController{},"get,post:ApiFunc")

Below is an example of different HTTP methods mapping to different controller methods. `;` as the separator:

	beego.Router("/simple",&SimpleController{},"get:GetFunc;post:PostFunc")

Below are the acceptable HTTP methods:

* *：including all methods below
* get ：GET request
* post ：POST request
* put ：PUT request
* delete ：DELETE request
* patch ：PATCH request
* options ：OPTIONS request
* head ：HEAD request

If * and other HTTP methods are used together, the HTTP method will be executed first. For example:

	beego.Router("/simple",&SimpleController{},"*:AllFunc;post:PostFunc")

The `PostFunc` rather than the `AllFunc` will be executed for POST requests.

The router of custom methods does not support RESTful behaviour by default which means if you set the router like `beego.Router("/api",&RestController{},"post:ApiFunc")` and the request method is `POST` then the `Post` method won't be executed by default.

## Auto matching

Firstly you need to register controller as an auto-router.

	beego.AutoRouter(&controllers.ObjectController{})

Then Beego will retrieve all the methods in that controller by reflection and you can call the related methods in this way:

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

Then you can get the extension name of the url by accessing `this.Ctx.Input.Param(":ext")`.

## Annotations
Beego version 1.3 introduced support for annotation routers. You don't need to register all the routers inside `router.go`. You only need to `Include` the controller. For example:

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

Then you can register the routers in `router.go`

    beego.Include(&CMSController{})

beego will parse the source code automatically. Notes: it will only be generated under dev mode.

Then the following routers will be supported:

* GET /staticblock/:key
* GET /all/:key

It's exactly same as registering by Router functions:

    beego.Router("/staticblock/:key", &CMSController{}, "get:StaticBlock")
    beego.Router("/all/:key", &CMSController{}, "get:AllBlock")

The `URLMapping` function above is a new function introduced in Beego 1.3. If you didn't use `URLMapping`, Beego will find the function by reflection otherwise Beego will find the function by `interface` which is much faster.

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
    We recommend that you use these methods beginning with `NS` as it is compatible with the gofmt tool.

- NSCond(cond namespaceCond)

    if the namespaceCond returns true this namespace will be run, otherwise it won't run

- NSBefore(filterList ...FilterFunc)
- NSAfter(filterList ...FilterFunc)

    For `BeforeRouter` and `FinishRouter` filters. One can register multiple filters.

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

    These are methods to set up routers which are the same as the basic routers.

- NSNamespace(prefix string, params ...innnerNamespace)

    Nested namespaces

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

The methods below are methods for the`*Namespace` object. It is not recommended. They have the same functionality as the methods with `NS`. But the methods list above with `NS` are more elegant and easier to read.

- Cond(cond namespaceCond)

	if the namespaceCond return true will run this namespace,unwise won't run

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


You can nest even more function:

```
//APIS
ns :=
	beego.NewNamespace("/api",
		//此处正式版时改为验证加密请求
		beego.NSCond(func(ctx *context.Context) bool {
			if ua := ctx.Input.Request.UserAgent(); ua != "" {
				return true
			}
			return false
		}),
		beego.NSNamespace("/ios",
			//CRUD Create(创建)、Read(读取)、Update(更新)和Delete(删除)
			beego.NSNamespace("/create",
				// /api/ios/create/node/
				beego.NSRouter("/node", &apis.CreateNodeHandler{}),
				// /api/ios/create/topic/
				beego.NSRouter("/topic", &apis.CreateTopicHandler{}),
			),
			beego.NSNamespace("/read",
				beego.NSRouter("/node", &apis.ReadNodeHandler{}),
				beego.NSRouter("/topic", &apis.ReadTopicHandler{}),
			),
			beego.NSNamespace("/update",
				beego.NSRouter("/node", &apis.UpdateNodeHandler{}),
				beego.NSRouter("/topic", &apis.UpdateTopicHandler{}),
			),
			beego.NSNamespace("/delete",
				beego.NSRouter("/node", &apis.DeleteNodeHandler{}),
				beego.NSRouter("/topic", &apis.DeleteTopicHandler{}),
			)
		),
	)

beego.AddNamespace(ns)
```
