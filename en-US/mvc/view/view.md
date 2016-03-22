---
name: Template Parsing
sort: 1
---

# Template Parsing

Beego uses Go's built-in package `html/template` as the template parser.  Upon startup, it will compile and cache the templates into a map for efficient rendering.

## Template Directory

The default template directory for Beego is `views`. Template files can be put into this directory and Beego will parse and cache them automatically. However if the development mode is enabled, Beego parses templates every time without caching. Beego can only have one template directory which can be customized:

	beego.ViewsPath = "myviewpath"

## Auto Rendering

You don't need to render and output templates manually. Beego will call Render automatically after finishing the method. You can disable auto rendering in the configuration file or in `main.go` if you don't need it.

In configuration file:

	autorender = false

In main.go:

	beego.AutoRender = false

## Template Tags

Go uses `{{` and `}}` as the default template tags. In the case that these tags conflict with other template tags as in AngularJS, we can use other tags. To do so, add these to the configuration:

	beego.TemplateLeft = "<<<"
	beego.TemplateRight = ">>>"

## Template Data

Template gets its data from `this.Data` in Controller. So for example if you need `{{.Content}}` in the template, you need to assign it in the Controller first:

	this.Data["Content"] = "value"

Different rendering types:

- struct

  Struct variable:

		type A struct{
			Name string
			Age  int
		}

  Assign value in the Controller:

		this.Data["a"]=&A{Name:"astaxie",Age:25}

  Render it in the template:

		the username is {{.a.Name}}
		the age is {{.a.Age}}

- map

  Assign value in the Controller:

		mp["name"]="astaxie"
		mp["nickname"] = "haha"
		this.Data["m"]=mp

  Render it in the template:

		the username is {{.m.name}}
		the username is {{.m.nickname}}

- slice

  Assign value in the Controller:

		ss :=[]string{"a","b","c"}
		this.Data["s"]=ss

  Render it in the template:

		{{range $key, $val := .s}}
		{{$key}}
		{{$val}}
	    {{end}}

## Template Name

> From version 1.6: this.TplNames is this.TplName

Beego uses Go's built-in template engine, so the syntax is same as Go.  To learn more about template see [Templates](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/07.4.md).

You can set the template name in Controller and Beego will find the template file under the viewpath and render it automatically. In the config below, Beego will find add.tpl under admin and render it.

```go
this.TplName = "admin/add.tpl"
```

Beego supports `.tpl` and `.html` file extensions by default. If you're using other extensions, you must set it in the configuration first:

```go
beego.AddTemplateExt("file_extension_you_need")
```

If `TplName` is not set in the Controller while `autorender` is enabled, Beego will use `TplName` as below by default:

```go
c.TplName = strings.ToLower(c.controllerName) + "/" + strings.ToLower(c.actionName) + "." + c.TplExt
```

It is Controller name + "/" + request method name + "." + template extension. So if the Controller name is `AddController`, request method is `POST` and the default template extension is `tpl`, Beego will render `/viewpath/addcontroller/post.tpl` template file.

## Layout Design

Beego supports layout design. For example, if in your application the main navigation and footer does not change and only the content part is different, you can use a layout like this:

```go
this.Layout = "admin/layout.html"
this.TplName = "admin/add.tpl"
```

In `layout.html` you must set a variable like this:

	{{.LayoutContent}}

Beego will parse the file named `TplName` and assign it to `LayoutContent` then render `layout.html`.

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

Here is the logic in the Controller:

```go
type BlogsController struct {
    beego.Controller
}

func (this *BlogsController) Get() {
    this.Layout = "layout_blog.tpl"
    this.TplName = "blogs/index.tpl"
    this.LayoutSections = make(map[string]string)
    this.LayoutSections["HtmlHead"] = "blogs/html_head.tpl"
    this.LayoutSections["Scripts"] = "blogs/scripts.tpl"
    this.LayoutSections["Sidebar"] = ""
}
```
## Another approach

We can also just specify the template the controller is going to use and let the template system handle the layout:

for example:

controller:

```go
this.TplName = "blog/add.tpl"
this.Data["SomeVar"] = "SomeValue"
this.Data["Title"] = "Add"
```

template add.tpl:

	{{ template "layout_blog.tpl" . }}
	{{ define "css" }}
    		<link rel="stylesheet" href="/static/css/current.css">
	{{ end}}


	{{ define "content" }}
    		<h2>{{ .Title }}</h2>
    		<p> This is SomeVar: {{ .SomeVar }}</p>
	{{ end }}
	
	{{ define "js" }}
		<script src="/static/js/current.js"></script>
	{{ end}}


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
     {{ template "css" . }}
</head>
<body>

    <div class="container">
        {{ template "content" . }}
    </div>
    <script type="text/javascript" src="http://code.jquery.com/jquery-2.0.3.min.js"></script>
    <script src="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/js/bootstrap.min.js"></script>
     {{ template "js" . }}
</body>
</html>
```


## renderform

Define struct:

```go
type User struct {
	Id    int         `form:"-"`
	Name  interface{} `form:"username"`
	Age   int         `form:"age,text,age:"`
	Sex   string
	Intro string `form:",textarea"`
}
```

* StructTag definition uses `form` as tag. It uses the same tags as [Parse Form](../controller/params.md#parse-to-struct). There are three optional params separated by ',':

  * The first param is `name` attribute of the form field. If empty, it will use `struct field name` as the value.
  * The second param is the form field type. If empty, it is assumed as `text`.
  * The third param is the tag of form field. If empty, it will use `struct field name` as the tag name.

* If the `form` tag only has one value, it is the `name` attribute of the form field. Except last value can be ignore all the other place must be separated by ','. E.g.: `form:",,username:"`

* To ignore a field there are two ways:
  * The first way is to use lowercase for the field name in the struct.
  * The second way is to set `-` as the value of `form` tag.

controller：

```go
func (this *AddController) Get() {
    this.Data["Form"] = &User{}
    this.TplName = "index.tpl"
}
```

The param of Form must be a pointer to a struct.

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
