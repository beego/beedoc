---
name: 路由设置
sort: 2
---

# 路由设置

什么时候路由设置呢？前面介绍的 MVC 结构执行时，介绍过 beego 存在三种方式的路由，固定路由、正则路由、自动路由，接下来详细的讲解如何使用这三种路由。

## RESTful 路由

在介绍这三种 beego 的路由实现之前先介绍 RESTful，我们知道 RESTful 是一种目前 API 开发中广泛采用的形式，beego 默认就是支持这样的请求方法，也就是用户 Get 请求就执行 Get 方法，Post 请求就执行 Post 方法。因此默认的路由是这样 RESTful 的请求方式。

## 固定路由

固定路由也就是全匹配的路由，如下所示：

	beego.Router("/", &controllers.MainController{})
	beego.Router("/admin", &admin.UserController{})
	beego.Router("/admin/index", &admin.ArticleController{})
	beego.Router("/admin/addpkg", &admin.AddController{})
	
如上所示的路由就是我们最常用的路由方式，一个固定的路由，一个控制器，然后根据用户请求方法不同请求控制器中对应的方法，典型的 RESTful 方式。	

## 正则路由

为了用户更加方便的路由设置，beego 参考了 sinatra 的路由实现，支持多种方式的路由：

- beego.Router(“/api/:id([0-9]+)“, &controllers.RController{})

	自定义正则匹配 //匹配 /api/123 :id= 123

- beego.Router(“/news/:all”, &controllers.RController{})

	全匹配方式 //匹配 /news/path/to/123.html :all= path/to/123.html

- beego.Router(“/user/:username([\w]+)“, &controllers.RController{})

	正则字符串匹配 //匹配 /user/astaxie :username = astaxie

- beego.Router(“/download/.”, &controllers.RController{})

	*匹配方式 //匹配 /download/file/api.xml :path= file/api :ext=xml

- beego.Router(“/download/ceshi/*“, &controllers.RController{})

	*全匹配方式 //匹配 /download/ceshi/file/api.json :splat=file/api.json

- beego.Router(“/:id:int”, &controllers.RController{})

	int 类型设置方式 //匹配 :id为int类型，框架帮你实现了正则([0-9]+)

- beego.Router(“/:hi:string”, &controllers.RController{})

	string 类型设置方式 //匹配 :hi为string类型。框架帮你实现了正则([\w]+)

可以在 Controlle 中通过如下方式获取上面的变量：

	this.Ctx.Input.Param(":id")
	this.Ctx.Input.Param(":username")
	this.Ctx.Input.Param(":splat")
	this.Ctx.Input.Param(":path")
	this.Ctx.Input.Param(":ext")

## 自定义方法及 RESTful 规则

上面列举的是默认的请求方法名（请求的 method 和函数名一致，例如 `GET` 请求执行 `Get` 函数，`POST` 请求执行 `Post` 函数），如果用户期望自定义函数名，那么可以使用如下方式：

	beego.Router("/",&IndexController{},"*:Index")
	
使用第三个参数，第三个参数就是用来设置对应 method 到函数名，定义如下

* `*`表示任意的 method 都执行该函数
* 使用 httpmethod:funcname 格式来展示
* 多个不同的格式使用 `;` 分割
* 多个 method 对应同一个 funcname，method 之间通过 `,` 来分割

以下是一个 RESTful 的设计示例：

	beego.Router("/api/list",&RestController{},"*:ListFood")
	beego.Router("/api/create",&RestController{},"post:CreateFood")
	beego.Router("/api/update",&RestController{},"put:UpdateFood")
	beego.Router("/api/delete",&RestController{},"delete:DeleteFood")
	
以下是多个 HTTP Method 指向同一个函数的示例：

	beego.Router("/api",&RestController{},"get,post:ApiFunc")
	
以下是不同的 method 对应不同的函数，通过 ; 进行分割的示例：

	beego.Router("/simple",&SimpleController{},"get:GetFunc;post:PostFunc")

可用的 HTTP Method：

* *：包含一下所有的函数
* get ：GET 请求
* post ：POST 请求
* put ：PUT 请求
* delete ：DELETE 请求
* patch ：PATCH 请求
* options ：OPTIONS 请求
* head ：HEAD 请求

如果同时存在 * 和对应的 HTTP Method，那么优先执行 HTTP Method 的方法，例如同时注册了如下所示的路由：

	beego.Router("/simple",&SimpleController{},"*:AllFunc;post:PostFunc")
那么执行 `POST` 请求的时候，执行 `PostFunc` 而不执行 `AllFunc`。

>>>自定义函数的路由默认不支持RESTful的方法，也就是如果你设置了`beego.Router("/api",&RestController{},"post:ApiFunc")` 这样的路由，如果请求的方法是`POST`，那么不会默认去执行`Post`函数。

## 自动匹配

用户首先需要把需要路由的控制器注册到自动路由中：

	beego.AutoRouter(&controllers.ObjectController{})
那么 beego 就会通过反射获取该结构体中所有的实现方法，你就可以通过如下的方式访问到对应的方法中：

	/object/login   调用 ObjectController 中的 Login 方法
	/object/logout  调用 ObjectController 中的 Logout 方法
除了前缀两个` /:controller/:method `的匹配之外，剩下的 url beego会帮你自动化解析为参数，保存在 `this.Ctx.Input.Param` 当中：

	/object/blog/2013/09/12  调用 ObjectController 中的 Blog 方法，参数如下：map[0:2013 1:09 2:12]
方法名在内部是保存了用户设置的，例如 Login，url 匹配的时候都会转化为小写，所以，`/object/LOGIN` 这样的 `url` 也一样可以路由到用户定义的 `Login` 方法中。

现在已经可以通过自动识别出来下面类似的所有url，都会把请求分发到 `controller` 的 `simple` 方法：

	/controller/simple
	/controller/simple.html
	/controller/simple.json
	/controller/simple.rss
	
可以通过 `this.Ctx.Input.Param(":ext")` 获取后缀名。
