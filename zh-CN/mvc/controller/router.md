---
name: 路由设置
sort: 2
---

# 路由设置

什么是路由设置呢？前面介绍的 MVC 结构执行时，介绍过 beego 存在三种方式的路由:固定路由、正则路由、自动路由，接下来详细的讲解如何使用这三种路由。

## 基础路由

从 beego 1.2 版本开始支持了基本的 RESTful 函数式路由,应用中的大多数路由都会定义在 `routers/router.go` 文件中。最简单的 beego 路由由 URI 和闭包函数组成。

### 基本 GET 路由

```
web.Get("/",func(ctx *context.Context){
     ctx.Output.Body([]byte("hello world"))
})
```

### 基本 POST 路由

```
web.Post("/alice",func(ctx *context.Context){
     ctx.Output.Body([]byte("bob"))
})
```

### 注册一个可以响应任何 HTTP 的路由

```
web.Any("/foo",func(ctx *context.Context){
     ctx.Output.Body([]byte("bar"))
})
```

### 所有的支持的基础函数如下所示

* web.Get(router, web.FilterFunc)
* web.Post(router, web.FilterFunc)
* web.Put(router, web.FilterFunc)
* web.Patch(router, web.FilterFunc)
* web.Head(router, web.FilterFunc)
* web.Options(router, web.FilterFunc)
* web.Delete(router, web.FilterFunc)
* web.Any(router, web.FilterFunc)

### 支持自定义的 handler 实现
有些时候我们已经实现了一些 rpc 的应用,但是想要集成到 beego 中,或者其他的 httpserver 应用,集成到 beego 中来.现在可以很方便的集成:

```
s := rpc.NewServer()
s.RegisterCodec(json.NewCodec(), "application/json")
s.RegisterService(new(HelloService), "")
web.Handler("/rpc", s)
```

`web.Handler(router, http.Handler)` 这个函数是关键,第一个参数表示路由 URI, 第二个就是你自己实现的 `http.Handler`, 注册之后就会把所有 rpc 作为前缀的请求分发到 `http.Handler` 中进行处理.

这个函数其实还有第三个参数就是是否是前缀匹配,默认是 false, 如果设置了 true, 那么就会在路由匹配的时候前缀匹配,即 `/rpc/user` 这样的也会匹配去运行
### 路由参数
后面会讲到固定路由,正则路由,这些参数一样适用于上面的这些函数

## RESTful Controller 路由

在介绍这三种 beego 的路由实现之前先介绍 RESTful，我们知道 RESTful 是一种目前 API 开发中广泛采用的形式，beego 默认就是支持这样的请求方法，也就是用户 Get 请求就执行 Get 方法，Post 请求就执行 Post 方法。因此默认的路由是这样 RESTful 的请求方式。

## 固定路由

固定路由也就是全匹配的路由，如下所示：

	web.Router("/", &controllers.MainController{})
	web.Router("/admin", &admin.UserController{})
	web.Router("/admin/index", &admin.ArticleController{})
	web.Router("/admin/addpkg", &admin.AddController{})

如上所示的路由就是我们最常用的路由方式，一个固定的路由，一个控制器，然后根据用户请求方法不同请求控制器中对应的方法，典型的 RESTful 方式。

## 正则路由

为了用户更加方便的路由设置，beego 参考了 sinatra 的路由实现，支持多种方式的路由：

- web.Router("/api/?:id", &controllers.RController{})

	默认匹配   //例如对于URL"/api/123"可以匹配成功，此时变量":id"值为"123"，URL"/api/"可正常匹配

- web.Router("/api/:id", &controllers.RController{})

	默认匹配   //例如对于URL"/api/123"可以匹配成功，此时变量":id"值为"123"，但URL"/api/"匹配失败

- web.Router("/api/:id([0-9]+)", &controllers.RController{})

	自定义正则匹配 //例如对于URL"/api/123"可以匹配成功，此时变量":id"值为"123"

- web.Router("/user/:username([\\\\w]+)", &controllers.RController{})

	正则字符串匹配 //例如对于URL"/user/astaxie"可以匹配成功，此时变量":username"值为"astaxie"

- web.Router("/download/\*.\*", &controllers.RController{})

	*匹配方式 //例如对于URL"/download/file/api.xml"可以匹配成功，此时变量":path"值为"file/api"， ":ext"值为"xml"

- web.Router("/download/ceshi/*", &controllers.RController{})

	*全匹配方式 //例如对于URL"/download/ceshi/file/api.json"可以匹配成功，此时变量":splat"值为"file/api.json"

- web.Router("/:id:int", &controllers.RController{})

	int 类型设置方式，匹配 :id为int 类型，框架帮你实现了正则 ([0-9]+)

- web.Router("/:hi:string", &controllers.RController{})

	string 类型设置方式，匹配 :hi 为 string 类型。框架帮你实现了正则 ([\w]+)

- web.Router("/cms_:id([0-9]+).html", &controllers.CmsController{})

	带有前缀的自定义正则 //匹配 :id 为正则类型。匹配 cms_123.html 这样的 url :id = 123



可以在 Controller 中通过如下方式获取上面的变量：

	this.Ctx.Input.Param(":id")
	this.Ctx.Input.Param(":username")
	this.Ctx.Input.Param(":splat")
	this.Ctx.Input.Param(":path")
	this.Ctx.Input.Param(":ext")

## 自定义方法及 RESTful 规则

上面列举的是默认的请求方法名（请求的 method 和函数名一致，例如 `GET` 请求执行 `Get` 函数，`POST` 请求执行 `Post` 函数），如果用户期望自定义函数名，那么可以使用如下方式：

	web.Router("/",&IndexController{},"*:Index")

使用第三个参数，第三个参数就是用来设置对应 method 到函数名，定义如下

* `*`表示任意的 method 都执行该函数
* 使用 httpmethod:funcname 格式来展示
* 多个不同的格式使用 `;` 分割
* 多个 method 对应同一个 funcname，method 之间通过 `,` 来分割

以下是一个 RESTful 的设计示例：

	web.Router("/api/food",&RestController{},"get:ListFood")
	web.Router("/api/food",&RestController{},"post:CreateFood")
	web.Router("/api/food",&RestController{},"put:UpdateFood")
	web.Router("/api/food",&RestController{},"delete:DeleteFood")

以下是多个 HTTP Method 指向同一个函数的示例：

	web.Router("/api",&RestController{},"get,post:ApiFunc")

以下是不同的 method 对应不同的函数，通过 ; 进行分割的示例：

	web.Router("/api/food",&RestController{},"get:ListFood;post:CreateFood;put:UpdateFood;delete:DeleteFood")

可用的 HTTP Method：

* *: 包含以下所有的函数
* get: GET 请求
* post: POST 请求
* put: PUT 请求
* delete: DELETE 请求
* patch: PATCH 请求
* options: OPTIONS 请求
* head: HEAD 请求

如果同时存在 * 和对应的 HTTP Method，那么优先执行 HTTP Method 的方法，例如同时注册了如下所示的路由：

	web.Router("/simple",&SimpleController{},"*:AllFunc;post:PostFunc")
那么执行 `POST` 请求的时候，执行 `PostFunc` 而不执行 `AllFunc`。

>>>自定义函数的路由默认不支持 RESTful 的方法，也就是如果你设置了 `web.Router("/api",&RestController{},"post:ApiFunc")` 这样的路由，如果请求的方法是 `POST`，那么不会默认去执行 `Post` 函数。

## 自动匹配

用户首先需要把需要路由的控制器注册到自动路由中：

	web.AutoRouter(&controllers.ObjectController{})
那么 beego 就会通过反射获取该结构体中所有的实现方法，你就可以通过如下的方式访问到对应的方法中：

	/object/login   调用 ObjectController 中的 Login 方法
	/object/logout  调用 ObjectController 中的 Logout 方法
除了前缀两个 `/:controller/:method` 的匹配之外，剩下的 url beego 会帮你自动化解析为参数，保存在 `this.Ctx.Input.Params` 当中：

	/object/blog/2013/09/12  调用 ObjectController 中的 Blog 方法，参数如下：map[0:2013 1:09 2:12]
方法名在内部是保存了用户设置的，例如 Login，url 匹配的时候都会转化为小写，所以，`/object/LOGIN` 这样的 `url` 也一样可以路由到用户定义的 `Login` 方法中【实际在1.10.1版本中测试，不会转化大小写，即只能匹配`/object/login`，其他均不能正常识别】。

现在已经可以通过自动识别出来下面类似的所有 url，都会把请求分发到 `controller` 的 `simple` 方法：

	/controller/simple
	/controller/simple.html
	/controller/simple.json
	/controller/simple.xml

可以通过 `this.Ctx.Input.Param(":ext")` 获取后缀名。

## 注解路由

从2.0开始，我们使用配置`CommentRouterPath`来配置注解路由的扫描路径。在`dev`环境下，我们将自动扫描该配置指向的目录及其子目录，生成路由文件。

生成之后，用户需要显示 Include 相应地 controller。注意， controller 的 method 方法上面须有 router 注释（// @router），详细的使用请看下面的例子：

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

可以在 `router.go` 中通过如下方式注册路由：

	web.Include(&CMSController{})


web 自动会进行源码分析，注意只会在 dev 模式下进行生成，生成的路由放在 "/routers/commentsRouter.go" 文件中。

这样上面的路由就支持了如下的路由：

* GET /staticblock/:key
* GET /all/:key

其实效果和自己通过 Router 函数注册是一样的：

	web.Router("/staticblock/:key", &CMSController{}, "get:StaticBlock")
	web.Router("/all/:key", &CMSController{}, "get:AllBlock")

同时大家注意到新版本里面增加了 URLMapping 这个函数，这是新增加的函数，用户如果没有进行注册，那么就会通过反射来执行对应的函数，如果注册了就会通过 interface 来进行执行函数，性能上面会提升很多。

## 方法表达式路由
方法表达式路由与上面的RESTful基本相似，区别是无需在传入http method和controller方法（如：`"get:StaticBlock"`）。
只需要通过golang的method expression进行传入方法表达式。如果方法是receiver是非指针，则直接使用 `包名.Controller.Method` 方法 传入，
如果receiver是指针，则使用 `(*包名.Controller).Method` 进行传参。假如在同包下，包名可进行省略。

````golang
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

共有以下几种函数：

* web.RouterGet(router, pkg.controller.method)
* web.RouterPost(router, pkg.controller.method)
* web.RouterPut(router, pkg.controller.method)
* web.RouterPatch(router, pkg.controller.method)
* web.RouterHead(router, pkg.controller.method)
* web.RouterOptions(router, pkg.controller.method)
* web.RouterDelete(router, pkg.controller.method)
* web.RouterAny(router, pkg.controller.method)


同时也支持namespace的使用：

* web.NSRouterGet
* web.NSRouterPost
* ......


## namespace

```
//初始化 namespace
ns :=
web.NewNamespace("/v1",
	web.NSCond(func(ctx *context.Context) bool {
		if ctx.Input.Domain() == "api.beego.me" {
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
//注册 namespace
web.AddNamespace(ns)
```
上面这个代码支持了如下这样的请求 URL

* GET /v1/notallowed
* GET /v1/version
* GET /v1/changepassword
* POST /v1/changepassword
* GET /v1/shop/123
* GET /v1/cms/ 对应 MainController、CMSController、BlockController 中的注解路由

而且还支持前置过滤,条件判断,无限嵌套 namespace

namespace 的接口如下:

- NewNamespace(prefix string, funcs ...interface{})

	初始化 namespace 对象,下面这些函数都是 namespace 对象的方法,但是强烈推荐使用 NS 开头的相应函数注册，因为这样更容易通过 gofmt 工具看的更清楚路由的级别关系

- NSCond(cond namespaceCond)

	支持满足条件的就执行该 namespace, 不满足就不执行

- NSBefore(filiterList ...FilterFunc)
- NSAfter(filiterList ...FilterFunc)

	上面分别对应 beforeRouter 和 FinishRouter 两个过滤器，可以同时注册多个过滤器

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

	上面这些都是设置路由的函数,详细的使用和上面 beego 的对应函数是一样的

- NSNamespace(prefix string, params ...innnerNamespace)

	嵌套其他 namespace

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


下面这些函数都是属于 *Namespace 对象的方法：不建议直接使用，当然效果和上面的 NS 开头的函数是一样的，只是上面的方式更优雅，写出来的代码更容易看得懂

- Cond(cond namespaceCond)

	支持满足条件的就执行该 namespace, 不满足就不执行,例如你可以根据域名来控制 namespace

- Filter(action string, filter FilterFunc)

	action 表示你需要执行的位置, before 和 after 分别表示执行逻辑之前和执行逻辑之后的 filter

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

	上面这些都是设置路由的函数,详细的使用和上面 beego 的对应函数是一样的

- Namespace(ns ...*Namespace)

更多的例子代码：

```
//APIS
ns :=
	web.NewNamespace("/api",
		//此处正式版时改为验证加密请求
		web.NSCond(func(ctx *context.Context) bool {
			if ua := ctx.Input.Request.UserAgent(); ua != "" {
				return true
			}
			return false
		}),
		web.NSNamespace("/ios",
			//CRUD Create(创建)、Read(读取)、Update(更新)和Delete(删除)
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
