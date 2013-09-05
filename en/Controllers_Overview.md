# Controllers

Use `beego.controller` as anonymous in your controller struct to implement the interface in Beego:

```go
type xxxController struct {
	beego.Controller
}
```

`beego.Controller` implemented`beego.ControllerInterface`, `beego.ControllerInterface` defined following methods:

- Init(ct *Context, cn string)

	Initialize context, controller's name, template's name, and container of template arguments

- Prepare()

	This is for expend usages, it executes before all the following methods. Users can overload this method for verification for example.

- Get()

	This method executes when client sends request as GET method, 403 as default status code. Users overload this method for customized handle process of GET method.

- Post()

	This method executes when client sends request as POST method, 403 as default status code. Users overload this method for customized handle process of POST method.

- Delete()

	This method executes when client sends request as DELETE method, 403 as default status code. Users overload this method for customized handle process of DELETE method.

- Put()

	This method executes when client sends request as PUT method, 403 as default status code. Users overload this method for customized handle process of PUT method.

- Head()

	This method executes when client sends request as HEAD method, 403 as default status code. Users overload this method for customized handle process of HEAD method.

- Patch()

	This method executes when client sends request as PATCH method, 403 as default status code. Users overload this method for customized handle process of PATCH method.

- Options()

	This method executes when client sends request as OPTIONS method, 403 as default status code. Users overload this method for customized handle process of OPTIONS method.

- Finish()

	This method executes after corresponding method finished, empty as default. User overload this method for more usages like close database, clean data, etc.

- Render() error

	This method is for rendering template, it executes automatically when you set beego.AutoRender to true.

Overload all methods for all customized logic processes, let's see an example:

```go
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

## Handle request

We always need to get data from users, including methods like GET, POST, etc. Beego parses these data automatically, and you can access them by following code:

- GetString(key string) string
- GetInt(key string) (int64, error)
- GetBool(key string) (bool, error)

Usage example:

```go
func (this *MainController) Post() {
	jsoninfo := this.GetString("jsoninfo")
	if jsoninfo == "" {
		this.Ctx.WriteString("jsoninfo is empty")
		return
	}
}
```

If you need other types that are not included above, like you need int64 instead of int, then you need to do following way:

```go
func (this *MainController) Post() {
	id := this.Input().Get("id")
	intid, err := strconv.Atoi(id)
}
```

To use `this.Ctx.Request` for more information about request, and object properties and method please read [Request](http://gowalker.org/pkg/net/http#Request).

## File upload

It's very easy to upload file through Beego, but don't forget to add `enctype="multipart/form-data"` in your form, otherwise the browser will not upload anything.

Files will be saved in memory, if the size is greater than cache memory, the rest part will be saved as temporary file. The default cache memory is 64 MB, and you can use following ways to change this size.

In code:

	beego.MaxMemory = 1<<22

In configuration file:

	maxmemory = 1<<22

Beego provides two convenient functions to upload files:

- GetFile(key string) (multipart.File, `*`multipart.FileHeader, error)

	This function is mainly used to read file name element `the_file` in form and returns corresponding information. You can use this information either filter or save files.

- SaveToFile(fromfile, tofile string) error

	This function a wrapper of GetFile and gives ability to save file.

This is an example to save file that is uploaded:

```go
func (this *MainController) Post() {
	this.SaveToFile("the_file","/var/www/uploads/uploaded_file.txt")
}
```


## Output Json and XML

Beego considered API function design at the beginning, and we often use Json or XML format data as output. Therefore, it's no reason that Beego doesn't support it:

Set `content-type` to `application/json` for output raw Json format data:

```go
func (this *AddController) Get() {
	mystruct := { ... }
	this.Data["json"] = &mystruct
	this.ServeJson()
}
``

Set `content-type` to `application/xml` for output raw XML format data:

```go
func (this *AddController) Get() {
	mystruct := { ... }
	this.Data["xml"]=&mystruct
	this.ServeXml()
}
```


## Redirect and error

You can use following to redirect:

```go
func (this *AddController) Get() {
	this.Redirect("/", 302)
}
```

You can also throw an exception in your controller as follows:

```go
func (this *MainController) Get() {
	this.Abort("401")
	v := this.GetSession("asta")
	if v == nil {
		this.SetSession("asta", int(1))
		this.Data["Email"] = 0
	} else {
		this.SetSession("asta", v.(int)+1)
		this.Data["Email"] = v.(int)
	}
	this.TplNames = "index.tpl"
}
```

Then Beego will not execute rest code of the function body when you call `this.Abort("401")`, and gives following default page view to users:

![](https://raw.github.com/astaxie/beego/master/docs/zh/images/401.png)

Beego supports following error code: 404, 401, 403, 500 and 503, you can customize your error handle, for example, use following code to replace 404 error handle process:

```go
func page_not_found(rw http.ResponseWriter, r *http.Request) {
	t, _ := template.New("beegoerrortemp").ParseFiles(beego.ViewsPath + "/404.html")
	data := make(map[string]interface{})
	data["content"] = "page not found"
	t.Execute(rw, data)
}

func main() {
	beego.Errorhandler("404", page_not_found)
	beego.Router("/", &controllers.MainController{})
	beego.Run()
}
```

You may be able to use your own `404.html` for your 404 error.

Beego also gives you ability to modify error message that shows on the error page, the following example shows how to set more meaningful error message when database has problems:

```go
func dbError(rw http.ResponseWriter, r *http.Request) {
	t, _ := template.New("beegoerrortemp").ParseFiles(beego.ViewsPath + "/dberror.html")
	data := make(map[string]interface{})
	data["content"] = "database is now down"
	t.Execute(rw, data)
}

func main() {
	beego.Errorhandler("dbError", dbError)
	beego.Router("/", &controllers.MainController{})
	beego.Run()
}
```

After you registered this customized error, you can use `this.Abort("dbError")` for any database error in your applications.

## Handle response

There are some situations that you may have in response:

1. Output template

	I've already talked about template above, Beego outputs template after corresponding method executed.

2. Redirect

	You can use this.Redirect("/", 302) to redirect page.

3. Output string

	Sometimes we just need to print string on the screen:
	
		this.Ctx.WriteString("ok")
