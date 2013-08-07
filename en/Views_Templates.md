# Template

### Template directory

Beego uses `views` as the default directory for template files, parses and caches them as needed(cache is not enable in develop mode), but you can **change**(because only one directory can be used for template files) its directory using following code:

	beego.ViewsPath = "/myviewpath"

### Auto-render

You don't need to call render function manually, Beego calls it automatically after corresponding methods executed. If your application is somehow doesn't need templates, you can disable this feature either in code of `main.go` or configuration file.

To disable auto-render in configuration file:

	autorender = false

To disable auto-render in `main.go`(before you call `beego.Run()` to run the application):

	beego.AutoRender = false

### Template data

You can use `this.Data` in controller methods to access the data in templates. Suppose you want to get content of `{{.Content}}`, you can use following code to do this:

	this.Data["Content"] = "value"

### Template name

Beego uses built-in template engine of Go, so there is no different in syntax. As for how to write template file, please visit [Template tutorial](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/07.4.md).

Beego parses template files in `viewpath` and render it after you set the name of the template file in controller methods. For example, Beego finds the file `add.tpl` in directory `admin` in following code:

	this.TplNames = "admin/add.tpl"

Beego supports two kinds of extensions for template files, which are `tpl` and `html`, if you want to use other extensions, you have to use following code to let Beego know:

	beego.AddTemplateExt("<your template file extension>")

If you enabled auto-render and you don't tell Beego which template file you are going to use in controller methods, Beego uses following format to find the template file if it exists:

	c.TplNames = c.ChildName + "/" + c.Ctx.Request.Method + "." + c.TplExt

Which is `<corresponding controller name>/<request method name>.<template extension>`. For example, your controller name is `AddController` and the request method is POST, and the default file extension is `tpl`, so Beego will try to find file `/<viewpath>/AddController/POST.tpl`.

### Layout design

Beego supports layout design, which means if you are working on an administration application, and some part of its user interface is exactly same all the time, then you can make this part as a layout.

	this.Layout = "admin/layout.html"
	this.TplNames = "admin/add.tpl" 

You have to set following variable in order to make Beego possible to insert your dynamic content:

	{{.LayoutContent}}

Beego parses template file and assign content to `LayoutContent`, and render them together.

Right now, Beego caches all template files, so you can use following way to implement another kind of layout:

	{{template "header.html"}}
	Handle logic
	{{template "footer.html"}}


### Template function

Beego supports customized template functions that are registered before you call `beego.Run()`.

	func hello(in string)(out string){
		out = in + "world"
		return
	}
	
	beego.AddFuncMap("hi",hello)

Then you can use this function in your template files:

	{{.Content | hi}}

There are some built-in template functions:

* markdown

	This function converts markdown content to HTML format, use {{markdown .Content}} in template files.

* dateformat

	This function converts time to formatted string, use {{dateformat .Time "2006-01-02T15:04:05Z07:00"}} in template files.

* date

	This function implements date function like in PHP, use formatted string to get corresponding time, use {{date .T "Y-m-d H:i:s"}} in template files.

* compare

	This functions compares two objects, returns true if they are same, false otherwise, use {{compare .A .B}} in template files.

* substr

	This function cuts out string from another string by index, it supports UTF-8 characters, use {{substr .Str 0 30}} in template files.

* html2str

	This function escapes HTML to raw string, use {{html2str .Htmlinfo}} in template files.

* str2html

	This function outputs string in HTML format without escaping, use {{str2html .Strhtml}} in template files.

* htmlquote

	This functions implements basic HTML escape, use {{htmlquote .quote}} in template files.

* htmlunquote

	This functions implements basic invert-escape of HTML, use {{htmlunquote .unquote}} in template files.

### Static files

Go provides `http.ServeFile` for static files, Beego wrapped this function and use following way to register static file folder:

	beego.SetStaticPath("/static","public")

- The first argument is the path of your URL.
- The second argument is the directory in your application path.

Beego supports multiple static file directories as follows:

	beego.SetStaticPath("/images","images")
	beego.SetStaticPath("/css","css")
	beego.SetStaticPath("/js","js")

After you setting static directory, when users visit `/images/login/login.png`，Beego accesses `images/login/login.png` in related to your application directory. One more example, if users visit `/static/img/logo.png`, Beego accesses file `public/img/logo.png`.