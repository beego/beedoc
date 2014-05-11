---
name: Template Parsing
sort: 1
---

# Template Parsing

Beego uses Go's builtin package `html/template` as the template parser.  Upon startup, it will compile and cache the templates into a map for efficient rendering.

## Template Directory

The default template directory for Beego is `views`. Template files can be put in this directory and Beego will parse and cache them automatically. However if the development mode is enabled, Beego parses templates everytime without caching. Beego can only have one template directory which can be customized:

	beego.ViewsPath = "myviewpath"

## Auto Rendering

You don't need to render and output template manually. Beego will call Render automatically after finishing the method. You can disable auto rendering in configuration file or in `main.go` if you don't need it.

In configuration file:

	autorender = false

In main.go:

	beego.AutoRender = false

## Template Tags

Go uses `{{` and `}}` as the default template tags. In the case that these tags confilict with other template tags as in AngularJS, we can use other tags. To do so, add these to the configuration:

	beego.TemplateLeft = "<<<"
	beego.TemplateRight = ">>>"

## Template Data

Template gets its data from `this.Data` in Controller. So for example if you need `{{.Content}}` in template, you need to assign it in Controller first:

	this.Data["Content"] = "value"

Different rendering types:

- struct

  Struct variable:

		type A struct{
			Name string
			Age  int
		}

  Assign value in Controller:

		this.Data["a"]=&A{Name:"astaxie",Age:25}

  Render it in template:

		the username is {{.a.Name}}
		the age is {{.a.Age}}

- map

  Assign value in Controller:

		mp["name"]="astaxie"
		mp["nickname"] = "haha"
		this.Data["m"]=mp

  Render it in template:

		the username is {{.m.name}}
		the username is {{.m.nickname}}

- slice

  Assign value in Controller:

		ss :=[]string{"a","b","c"}
		this.Data["s"]=ss

  Render it in template:

		{{range $key, $val := .s}}
		{{$key}}
		{{$val}}
	    {{end}}

## Template Name

Beego uses Go's builtin template engine, so the syntax is same as Go.  To learn more about template see [Templates](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/ebook/07.4.md).

You can set template name in Controller and Beego will find the template file under the viewpath and render it automatically. In the config below, Beego will find add.tpl under admin and render it.

	this.TplNames = "admin/add.tpl"

Beego supports `.tpl` and `.html` file extensions by default. If you're using other extensions, you must set it in the configuration first:

	beego.AddTemplateExt("file_extension_you_needed")

If TplNames is not set in Controller while `autorender` is enabled, Beego will use TplNames below by default:

	c.TplNames = strings.ToLower(c.controllerName) + "/" + strings.ToLower(c.actionName) + "." + c.TplExt

It is Controller name + "/" + request method name + "." + template extension. So if the Controller name is `AddController`, request method is `POST` and the default template extension is `tpl`, Beego will render `/viewpath/addcontroller/post.tpl` template file.

## Layout Design

Beego supports layout design. For example, if in your application the main navigation and footer don't change and only the content part is different, you can use a layout like this:

	this.Layout = "admin/layout.html"
	this.TplNames = "admin/add.tpl"

In `layout.html` you must set a variable like this:

	{{.LayoutContent}}

Beego will parse file of `TplNames` and assign it to `LayoutContent` then render `layout.html`.

Beego will cache all the template files. You can also implement a layout this way:

	{{template "header.html"}}
	Logic code
	{{template "footer.html"}}

## LayoutSection

`LayoutContent` is a little complicated, as it can include Javascript and CSS. Since in most situations having only one `LayoutContent` is not enough, there is an attribute called `LayoutSections` in `Controller`. It allows us to set multiple `section` in `Layout` page and each `section` can contain its own sub-template page.

layout_blog.tpl:

```
<!DOCTYPE html>
<html>
<head>
    <title>Lin Li</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css">
    <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap-theme.min.css">
    {{.HtmlHead}}
</head>
<body>

    <div class="container">
        {{.LayoutContent}}
    </div>
    <div>
        {{.SideBar}}
    </div>
    <script type="text/javascript" src="http://code.jquery.com/jquery-2.0.3.min.js"></script>
    <script src="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/js/bootstrap.min.js"></script>
    {{.Scripts}}
</body>
</html>
```

html_head.tpl:

```
<style>
     h1 {
        color: red;
     }
</style>
```

scripts.tpl：

```
<script type="text/javascript">
    $(document).ready(function() {
        // bla bla bla
    });
</script>
```

Here is the logic in Controller:

```
type BlogsController struct {
    beego.Controller
}

func (this *BlogsController) Get() {
    this.Layout = "layout_blog.tpl"
    this.TplNames = "blogs/index.tpl"
    this.LayoutSections = make(map[string]string)
    this.LayoutSections["HtmlHead"] = "blogs/html_head.tpl"
    this.LayoutSections["Scripts"] = "blogs/scripts.tpl"
    this.LayoutSections["Sidebar"] = ""
}
```

## renderform

Define struct:

	type User struct {
		Id    int         `form:"-"`
		Name  interface{} `form:"username"`
		Age   int         `form:"age,text,age:"`
		Sex   string
		Intro string `form:",textarea"`
	}

* StructTag definition uses `form` as tag. It uses same tags as [Parse Form](../controller/params.md#parse-to-struct). There are three optional params separated by ',':

  * The first param is `name` attribute of the form field. If empty, will use `struct field name` as the value.
  * The second param is the form field type. If empty, it is assumed as `text`.
  * The third param is the tag of form field. If empty, will use `struct field name` as the tag name.

* If the `form` tag only has one value, it is the `name` attribute of the form field. Except last value can be ignore all the other place must be separated by ','. E.g.: `form:",,username:"`

* To ignore a field there are two ways:
  * The first one is to use lowercase for the field name in the struct.
  * The second one is to set `-` as the value of `form` tag.

controller：

	func (this *AddController) Get() {
	    this.Data["Form"] = &User{}
	    this.TplNames = "index.tpl"
	}

The param of Form must be a pointor to a struct.

template:

	<form action="" method="post">
	{{.Form | renderform}}
	</form>

The code above will generate the form below:

```
	Name: <input name="username" type="text" value="test"></br>
	Age: <input name="age" type="text" value="0"></br>
	Gender: <input name="Sex" type="text" value=""></br>
	Intro: <input name="Intro" type="textarea" value="">
```
