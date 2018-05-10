---
name: Controller funcs
sort: 3
---

# Introduction to controller

> Note: From version 1.6: `this.ServeJson()` has been changed to `this.ServeJSON()` and `this.TplNames` has been changed to `this.TplName`

Beego's controller needs to be embeded as `beego.Controller`:

	type xxxController struct {
	    beego.Controller
	}

`beego.Controller` implements interface `beego.ControllerInterface`.  `beego.ControllerInterface` defines these functions:

- Init(ct *context.Context, controllerName, actionName string, app interface{})

  This function will initialize Context, Controller name, template name, template variable container `Data`. `app` is the executing Controller's reflecttype. This can be used to execute the subclass's methods.

- Prepare()

  This function is used for extension and will execute before the methods below. It can be overwritten to implement functions such as user validation.

- Get()

  This method will be executed if the HTTP request method is GET. It returns 403 by default. This can be used to handle GET requests by overwriting them in the struct of subclass.

- Post()

  This method will be executed if the HTTP request method is POST.  It returns 403 by default. This can be used to handle POST requests by overwriting them in the struct of subclass.

- Delete()

  This method will be executed if the HTTP request method is DELETE.  It returns 403 by default. This can be used to handle DELETE requests by overwriting them in the struct of subclass.

- Put()

  This method will be executed if the HTTP request method is PUT. It returns 403 by default. This can be used to handle PUT requests by overwriting them in the struct of subclass.

- Head()

  This method will be executed if the HTTP request method is HEAD. It return 403 by default. This can be used to handle HEAD requests by overwriting them in the struct of subclass.

- Patch()

  This method will be executed if the HTTP request method is PATCH. It returns 403 by default. This can be used to handle PATCH requests by overwriting them in the struct of subclass.

- Options()

  This method will be executed if the HTTP request method is OPTIONS. It returns 403 by default. This can be used to handle OPTIONS requests by overwriting them in the struct of subclass.

- Finish()

  This method will be executed after finishing the related HTTP method. It is empty by default. This can be implemented by overwriting it in the struct of subclass. It is used for database closing, data cleaning and so on.

- Render() error

  This method is used to render templates. It is only executed if `beego.AutoRender` is set to true.

Custom logica can be implemented by overwriting functions in struct. For example:

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

In the example above a RESTful structure has been implemented by overwriting functions.

The following example implements a baseController and other initialization methods that will be inherited by other controllers:

```
type NestPreparer interface {
        NestPrepare()
}

// baseRouter implements global settings for all other routers.
type baseRouter struct {
        beego.Controller
        i18n.Locale
        user    models.User
        isLogin bool
}
// Prepare implements Prepare method for baseRouter.
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

The above example defines a base class and initializes some variables. It will test if the executing Controller is an implementation of NestPreparer. If it is it calls the method of subclass. 

The example below shows an implementation of `NestPreparer`:

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

The above example first executes `Prepare`. Go will search for methods in the struct by looking in the parent classes. `BaseAdminRouter` will execute and checks whether there is a `Prepare` method. If not it keeps searching `baseRouter`. If there is it will execute the logic. `this.AppController` in `baseRouter` is the currently executing Controller `BaseAdminRouter`. Next, it will execute `BaseAdminRouter.NestPrepare` method. Finally, it will start executing the related `GET` or `POST` method.


## Stop controller executing immediately

To stop the execution logic of a request and return the response immediately use `StopRun()`. For example, when a user authentication fails in `Prepare` method a response will be returned immediately. 

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

>>> If you call `StopRun` the `Finish` method won't be run. To free resources call `Finish` manually before calling `StopRun`.


## Using PUT method in HTTP form

Since XHTML 1.x forms only support GET and POST these are the only allowed values for the "method" attribute. Optionally, this can be extended as follows:

Add a hidden input in the post form:

```
<form method="post" ...>
  <input type="hidden" name="_method" value="put" />
```

Then add a filter in Beego to check if the requests should be treated as a put request:

```go
var FilterMethod = func(ctx *context.Context) {
    if ctx.Input.Query("_method")!="" && ctx.Input.IsPost(){
          ctx.Request.Method = ctx.Input.Query("_method")
    }
}

beego.InsertFilter("*", beego.BeforeRouter, FilterMethod)
```
