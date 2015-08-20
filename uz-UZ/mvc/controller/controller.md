---
name: Controller funcs
sort: 3
---

# Introduction to controller

Based on the design of beego's controller, you just need to embed the `beego.Controller`:

	type xxxController struct {
	    beego.Controller
	}

`beego.Controller` implements interface `beego.ControllerInterface`.  `beego.ControllerInterface` defines these functions:

- Init(ct *context.Context, childName string, app interface{})

  This function will initialize Context, Controller name, template name, template variable container `Data`. `app` is the executing Controller's reflecttype. It can be used to execute subclass's methods.

- Prepare()

  You can use this function for extension, it will execute before the methods below. You can overwrite it to implement functions such as user validation.

- Get()

  This method will be executed if the HTTP request method is GET, return 403 by default. You can implement this method to handle GET request by overwriting it in the struct of subclass.

- Post()

  This method will be executed if the HTTP request method is POST, return 403 by default. You can implement this method to handle POST request by overwriting it in the struct of subclass.

- Delete()

  This method will be executed if the HTTP request method is DELETE, return 403 by default. You can implement this method to handle DELETE request by overwriting it in the struct of subclass.

- Put()

  This method will be executed if the HTTP request method is PUT, return 403 by default. You can implement this method to handle PUT request by overwriting it in the struct of subclass.

- Head()

  This method will be executed if the HTTP request method is HEAD, return 403 by default. You can implement this method to handle HEAD request by overwriting it in the struct of subclass.

- Patch()

  This method will be executed if the HTTP request method is PATCH, return 403 by default. You can implement this method to handle PATCH request by overwriting it in the struct of subclass.

- Options()

  This method will be executed if the HTTP request method is OPTIONS, return 403 by default. You can implement this method to handle OPTIONS request by overwriting it in the struct of subclass.

- Finish()

  This method will be executed after finishing related HTTP method, it's empty by default. You can implement this method by overwriting it in the struct of subclass. Used for database closing, data cleaning and so on.

- Render() error

  This method is used to render template. Only executed if `beego.AutoRender` is true.

By overwriting functions in struct, you can implement your own logic. For example:

```
type AddController struct {
    beego.Controller
}

func (this *AddController) Prepare() {

}

func (this *AddController) Get() {
    this.Data["content"] = "value"
    this.Layout = "admin/layout.html"
    this.TplNames = "admin/add.tpl"
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

In the example above, it implemented RESTful structure by overwriting functions.

Now let's see a popular architecture: Implementing a baseController and some initializing methods then other controllers just inherit from it.

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

The above example defined base class and initialized some variables. It will test if the executing Controller is an implementing of NestPreparer, if it is, then call the method of subclass. Let's see the implementation of `NestPreparer`:

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
	this.TplNames = "Get.tpl"
}

func (this *BaseAdminRouter) Post(){
	this.TplNames = "Post.tpl"
}
```

Here is our logic of it: executes `Prepare` first. It is the order of Go to find methods in struct, finding towards parent classes one by one. While execute `BaseAdminRouter`, checks if it has `Prepare` method. If no, keep finding `baseRouter`. If yes, it will execute the logic and `this.AppController` in `baseRouter` is the current executing Controller `BaseAdminRouter`. Then it will execute `BaseAdminRouter.NestPrepare` method. Then it will start executing related `GET` or `POST` method.
