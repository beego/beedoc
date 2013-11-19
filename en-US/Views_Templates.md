# Template

### Template directory

Beego uses `views` as the default directory for template files. Beego will parse and cache templates as needed (cache is not enable in develop mode). You can **change** the path views are loaded from - only one directory can be used for template files - using the following code:

```go
beego.ViewsPath = "/myviewpath"
```

### Auto-render

You don't need to call render function manually, Beego calls it automatically after the corresponding method is executed. If your application doesn't need templates for some reason, you can disable auto-rendering either in `main.go` or your configuration file.

To disable auto-render in configuration file:

	autorender = false

To disable auto-render in `main.go` (before you call `beego.Run()` to run the application):

```go
beego.AutoRender = false
```

### Template data

You use `this.Data` in controller methods to pass data into templates. Suppose you want to get content of `{{.Content}}`. In your controller you could do the following to set `Content`.:

```go
this.Data["Content"] = "value"
```

### Template name

Beego uses the template engine from Go's standard library, so there is no special syntax. As for how to write template file, please visit [Template tutorial](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/07.4.md).

Beego parses template files in `viewpath` and render it after you set the name of the template file in controller methods. For example, Beego finds the file `add.tpl` in directory `admin` in following code:

```go
this.TplNames = "admin/add.tpl"
```

Beego supports two kinds of extensions for template files, which are `tpl` and `html`, if you want to use other extensions, you have to use following code to let Beego know:

```go
beego.AddTemplateExt("<your template file extension>")
```

If you enabled auto-render and you don't tell Beego which template file you are going to use in controller methods, Beego uses following format to find the template file if it exists:

```go
c.TplNames = c.ChildName + "/" + c.Ctx.Request.Method + "." + c.TplExt
```

Which is `<corresponding controller name>/<request method name>.<template extension>`. For example, your controller name is `AddController` and the request method is POST, and the default file extension is `tpl`, so Beego will try to find file `/<viewpath>/AddController/post.tpl`.

### Layout design

Beego supports rendering layout designs. This allows you to separate common interface elements that wrap your views. For example, a content management system might have the same header, navigation, and footer on each page.

```go
this.Layout = "admin/layout.html"
this.TplNames = "admin/add.tpl" 
```

In your layout, you have to set following variable in order for Beego to insert your view's content:

	{{.LayoutContent}}

After rendering the controller tempalte, Beego will store the output into `LayoutContent` and then render the layout.

Right now, Beego caches all template files, so you can use following way to implement another kind of layout:

	{{template "header.html"}}
	Handle logic
	{{template "footer.html"}}


### Template functions

Beego supports customized template functions that are registered before you call `beego.Run()`.

```go
func hello(in string)(out string){
	out = in + "world"
	return
}

beego.AddFuncMap("hi", hello)
```

You can then use your registered function in your template files:

	{{.Content | hi}}

There are several built-in template functions:

* dateformat

	This function converts time to formatted string, use `{{dateformat .Time "2006-01-02T15:04:05Z07:00"}}` in template files.

* date

	This function provides date formatting similar to [PHP's date() function](http://php.net/date). By providing a formatting string, you can get the correctly formatted datetime string. For example, use `{{date .Time "Y-m-d H:i:s"}}` in template files.

* compare

	This functions compares two objects, returns true if they are same, false otherwise. Use `{{compare .A .B}}` in template files.

* substr

	This function cuts a string out of another string by index. It is UTF-8 aware, use `{{substr .Str 0 30}}` in template files.

* html2str

	This function escapes HTML to raw string. Use `{{html2str .Htmlinfo}}` in template files.

* str2html

	This function outputs string in HTML format without escaping. Use `{{str2html .Strhtml}}` in template files.

* htmlquote

	This functions implements basic HTML escaping, use `{{htmlquote .quote}}` in template files.

* htmlunquote

	This functions implements basic HTML decoding, use `{{htmlunquote .unquote}}` in template files.

### Static files

Go provides `http.ServeFile` for static files, Beego has wrapped this function and provides the following way to register a static file folder:

```go
beego.SetStaticPath("/static","public")
```

- The first argument is the path of your URL.
- The second argument is the directory in your application path.

Beego supports multiple static file directories as follows:

```go
beego.SetStaticPath("/images","images")
beego.SetStaticPath("/css","css")
beego.SetStaticPath("/js","js")
```

After you have set the static directories, when a user requests `/images/login/login.png`, Beego accesses `images/login/login.png`  relative to your application's directory. One more example, if a user requests `/static/img/logo.png`, Beego accesses file `public/img/logo.png`.
