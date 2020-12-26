---
name: 控制器函数
sort: 3
---

# 控制器介绍

基于 beego 的 Controller 设计，只需要匿名组合 `beego.Controller` 就可以了，如下所示：

```go

package your_package

import (
	"github.com/beego/beego/v2/server/web"
)

type xxxController struct {
	    web.Controller
}

```

## 控制器方法

`web.Controller` 实现了接口 `web.ControllerInterface`，`web.ControllerInterface` 定义了如下函数：

- Init(ctx *context.Context, controllerName, actionName string, app interface{})

	这个函数主要初始化了 Context、相应的 Controller 名称，模板名，初始化模板参数的容器 Data，app 即为当前执行的 Controller 的 reflecttype，这个 app 可以用来执行子类的方法。

- Prepare()

	这个函数主要是为了用户扩展用的，这个函数会在下面定义的这些 Method 方法之前执行，用户可以重写这个函数实现类似用户验证之类。

- Get()

	如果用户请求的 HTTP Method 是 GET，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Get 请求。

- Post()

	如果用户请求的 HTTP Method 是 POST，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Post 请求。

- Delete()

	如果用户请求的 HTTP Method 是 DELETE，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Delete 请求。

- Put()

	如果用户请求的 HTTP Method 是 PUT，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Put 请求.

- Head()

	如果用户请求的 HTTP Method 是 HEAD，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Head 请求。

- Patch()

	如果用户请求的 HTTP Method 是 PATCH，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Patch 请求.

- Options()

	如果用户请求的HTTP Method是OPTIONS，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Options 请求。

- Finish()

	这个函数是在执行完相应的 HTTP Method 方法之后执行的，默认是空，用户可以在子 struct 中重写这个函数，执行例如数据库关闭，清理数据之类的工作。

- Trace() error
    
    如果用户请求的 HTTP Method 是 Trace，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Head 请求。

- Render() error

	这个函数主要用来实现渲染模板，如果 beego.AutoRender 为 true 的情况下才会执行。

- Mapping(method string, fn func())
    
    注册一个方法。一般而言， method 是合法的 HTTP 方法名。当然，用户注册自己特定的业务逻辑方法，而后手动调用。
    
- HandlerFunc(fnname string) bool

    在前面 Mapping 方法里面注册的方法，可以通过该方法来使用。只会返回调用是否成功的信息——一般而言，只有方法不存在才会返回 false

- RenderBytes() ([]byte, error)
    
    将模板渲染成字节数组。需要注意的是，该方法并未检测`EnableRender`设置。并且，和`Render`方法相比，它并未将结果输出到`Response`。

- RenderString() (string, error)
    
    类似于`RenderBytes`方法。只是将结果转化为了`string`。
    
- Redirect(url string, code int)

    重定向。`url`是目的地址。

- SetData(data interface{})
    
    将`data`存储在控制的数据中。一般而言，你不会考虑用到这个方法。
    
- Abort(code string)
    
    中断当前方法的执行，直接返回该状态码，类似于`CustomAbort`。参考[errors](errors.md)
    
- CustomAbort(status int, body string)

    中断方法执行，直接返回该状态码和信息。参考[errors](errors.md)
    
- StopRun()
    
    直接触发`panic`。
    
- ServeXXX(encoding ...bool) error

    返回特性类型的响应。目前我们支持 JSON，JSONP，XML，YAML。参考[输出格式](jsonxml.md)
    
- ServeFormatted(encoding ...bool) error

    返回响应。其格式由客户端的`Accept`选项指定。参考[输出格式](jsonxml.md)

- Input() (url.Values, error)
    
    返回传入的参数。
    
- ParseForm(obj interface{}) error

    将表单反序列化到 obj 对象中。
    
- GetXXX(key string, def...) XXX, err

    从传入参数中，读取某个值。如果传入了默认值，那么在读取不到的情况下，返回默认值，否则返回错误。XXX 可以是 golang 所支持的基本类型，或者是 string, File 对象
    
- SaveToFile(fromfile, tofile string) error
    
    将上传的文件保存到文件系统中。其中`fromfile`是上传的文件的名字。

- SetSession(name interface{}, value interface{}) error

    往`Session`中设置值。
    
- GetSession(name interface{}) interface{}

    从`Session`中读取值。
    
- DelSession(name interface{}) error

    从`Session`中删除某项。
    
- SessionRegenerateID() error
    
    重新生成一个`SessionId`。
    
- DestroySession() error
    
    销毁`Session`
    
- IsAjax() bool

    是否是 Ajax 请求
    
- GetSecureCookie(Secret, key string) (string, bool)

    从`Cookie`中读取数据。`bool`返回值，表达是否取到了数据。
    
- SetSecureCookie(Secret, name, value string, others ...interface{})

    设置`Cookie`。
    
- XSRFToken() string
    
    创建一个`CSRF` token.
    
- CheckXSRFCookie() bool

    检测是否有`CSRF` token

## 子类扩展

通过子 struct 的方法重写，用户就可以实现自己的逻辑，接下来我们看一个实际的例子：

```go
type AddController struct {
    web.Controller
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

从上面的例子可以看出来，通过重写方法可以实现对应 method 的逻辑，实现 RESTFul 结构的逻辑处理。

下面我们再来看一种比较流行的架构，首先实现一个自己的基类 baseController，实现一些初始化的方法，然后其他所有的逻辑继承自该基类：

```go

type NestPreparer interface {
        NestPrepare()
}

// baseRouter implemented global settings for all other routers.
type baseController struct {
        web.Controller
        i18n.Locale
        user    models.User
        isLogin bool
}
// Prepare implemented Prepare method for baseRouter.
func (this *baseController) Prepare() {

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

```go
type BaseAdminRouter struct {
    baseController
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

这样我们的执行器执行的逻辑是这样的，首先执行 Prepare，这个就是 Go 语言中 struct 中寻找方法的顺序，依次往父类寻找。执行 `BaseAdminRouter` 时，查找他是否有 `Prepare` 方法，没有就寻找 `baseController`，找到了，那么就执行逻辑，然后在 `baseController` 里面的 `this.AppController` 即为当前执行的控制器 `BaseAdminRouter`，因为会执行 `BaseAdminRouter.NestPrepare` 方法。然后开始执行相应的 Get 方法或者 Post 方法。

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
