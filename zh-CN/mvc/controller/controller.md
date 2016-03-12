---
name: 控制器函数
sort: 3
---

# 控制器介绍

>> 提示：在v1.6中，此文档所涉及的API有重大变更，`this.ServeJson()` 更改为 `this.ServeJSON()`，`this.TplNames` 更改为 `this.TplName`。

基于 beego 的 Controller 设计，只需要匿名组合 `beego.Controller` 就可以了，如下所示：

	type xxxController struct {
	    beego.Controller
	}
`beego.Controller` 实现了接口 `beego.ControllerInterface`，`beego.ControllerInterface` 定义了如下函数：

- Init(ct *context.Context, childName string, app interface{})

	这个函数主要初始化了 Context、相应的 Controller 名称，模板名，初始化模板参数的容器 Data，app即为当前执行的Controller的reflecttype，这个app可以用来执行子类的方法。

- Prepare()

	这个函数主要是为了用户扩展用的，这个函数会在下面定义的这些 Method 方法之前执行，用户可以重写这个函数实现类似用户验证之类。

- Get()

	如果用户请求的 HTTP Method 是 GET，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Get 请求。

- Post()

	如果用户请求的 HTTP Method 是 POST，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Post 请求。

- Delete()

	如果用户请求的 HTTP Method 是 DELETE，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Delete 请求。

- Put()

	如果用户请求的 HTTP Method 是 PUT，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Put 请求.

- Head()

	如果用户请求的 HTTP Method 是 HEAD，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Head 请求。

- Patch()

	如果用户请求的 HTTP Method 是 PATCH，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Patch 请求.

- Options()

	如果用户请求的HTTP Method是OPTIONS，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Options 请求。

- Finish()

	这个函数实在执行完相应的 HTTP Method 方法之后执行的，默认是空，用户可以在子 struct 中重写这个函数，执行例如数据库关闭，清理数据之类的工作。

- Render() error

	这个函数主要用来实现渲染模板，如果 beego.AutoRender 为 true 的情况下才会执行。

所以通过子 struct 的方法重写，用户就可以实现自己的逻辑，接下来我们看一个实际的例子：

```
type AddController struct {
    beego.Controller
}

func (this *AddController) Prepare() {

}

func (this *AddController) Get() {
    this.Data["content"] = "value"
    this.Layout = "admin/layout.html"
    this.TplName = "admin/add.tpl"
}

func (this *AddController) Post() {
    pkgname := this.GetString("pkgname")
    content := this.GetString("content")
    pk := models.GetCruPkg(pkgname)
    if pk.Id == 0 {
        var pp models.PkgEntity
        pp.Pid = 0
        pp.Pathname = pkgname
        pp.Intro = pkgname
        models.InsertPkg(pp)
        pk = models.GetCruPkg(pkgname)
    }
    var at models.Article
    at.Pkgid = pk.Id
    at.Content = content
    models.InsertArticle(at)
    this.Ctx.Redirect(302, "/admin/index")
}
```

从上面的例子可以看出来，通过重写方法可以实现对应 method 的逻辑，实现 RESTful 结构的逻辑处理。

下面我们再来看一种比较流行的架构，首先实现一个自己的基类 baseController，实现一些初始化的方法，然后其他所有的逻辑继承自该基类：

```
type NestPreparer interface {
        NestPrepare()
}

// baseRouter implemented global settings for all other routers.
type baseRouter struct {
        beego.Controller
        i18n.Locale
        user    models.User
        isLogin bool
}
// Prepare implemented Prepare method for baseRouter.
func (this *baseRouter) Prepare() {
        
        // page start time
        this.Data["PageStartTime"] = time.Now()

        // Setting properties.
        this.Data["AppDescription"] = utils.AppDescription
        this.Data["AppKeywords"] = utils.AppKeywords
        this.Data["AppName"] = utils.AppName
        this.Data["AppVer"] = utils.AppVer
        this.Data["AppUrl"] = utils.AppUrl
        this.Data["AppLogo"] = utils.AppLogo
        this.Data["AvatarURL"] = utils.AvatarURL
        this.Data["IsProMode"] = utils.IsProMode

        if app, ok := this.AppController.(NestPreparer); ok {
                app.NestPrepare()
        }
}
```

上面定义了基类，大概是初始化了一些变量，最后有一个 Init 函数中那个 app 的应用，判断当前运行的 Controller 是否是 NestPreparer 实现，如果是的话调用子类的方法，下面我们来看一下 NestPreparer 的实现：

```
type BaseAdminRouter struct {
    baseRouter
}

func (this *BaseAdminRouter) NestPrepare() {
    if this.CheckActiveRedirect() {
            return
    }

    // if user isn't admin, then logout user
    if !this.user.IsAdmin {
            models.LogoutUser(&this.Controller)

            // write flash message
            this.FlashWrite("NotPermit", "true")

            this.Redirect("/login", 302)
            return
    }

    // current in admin page
    this.Data["IsAdmin"] = true

    if app, ok := this.AppController.(ModelPreparer); ok {
            app.ModelPrepare()
            return
    }
}

func (this *BaseAdminRouter) Get(){
	this.TplName = "Get.tpl"
}

func (this *BaseAdminRouter) Post(){
	this.TplName = "Post.tpl"
}
```

这样我们的执行器执行的逻辑是这样的，首先执行 Prepare，这个就是 Go 语言中 struct 中寻找方法的顺序，依次往父类寻找。执行 `BaseAdminRouter` 时，查找他是否有 `Prepare` 方法，没有就寻找 `baseRouter`，找到了，那么就执行逻辑，然后在 `baseRouter` 里面的 `this.AppController` 即为当前执行的控制器 `BaseAdminRouter`，因为会执行 `BaseAdminRouter.NestPrepare` 方法。然后开始执行相应的 Get 方法或者 Post 方法。

## 提前终止运行

我们应用中经常会遇到这样的情况，在 Prepare 阶段进行判断，如果用户认证不通过，就输出一段信息，然后直接中止进程，之后的 Post、Get 之类的不再执行，那么如何终止呢？可以使用 `StopRun` 来终止执行逻辑，可以在任意的地方执行。

```
type RController struct {
    beego.Controller
}

func (this *RController) Prepare() {
    this.Data["json"] = map[string]interface{}{"name": "astaxie"}
    this.ServeJSON()
    this.StopRun()
}
```

>>> 调用 StopRun 之后，如果你还定义了 Finish 函数就不会再执行，如果需要释放资源，那么请自己在调用 StopRun 之前手工调用 Finish 函数。


## 在表单中使用 PUT 方法

首先要说明, 在 XHTML 1.x 标准中, 表单只支持 GET 或者 POST 方法. 虽然说根据标准, 你不应该将表单提交到 PUT 方法, 但是如果你真想的话, 也很容易, 通常可以这么做:

首先表单本身还是使用 POST 方法提交, 但是可以在表单中添加一个隐藏字段:

```
<form method="post" ...>
  <input type="hidden" name="_method" value="put" />
```

接着在 Beego 中添加一个过滤器来判断是否将请求当做 PUT 来解析:

```go
var FilterMethod = func(ctx *context.Context) {
    if ctx.BeegoInput.Query("_method")!="" && ctx.BeegoInput.IsPost(){
          ctx.Request.Method = ctx.BeegoInput.Query("_method")
    }
}

beego.InsertFilter("*", beego.BeforeRouter, FilterMethod)
```
