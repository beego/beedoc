---
name: 模板处理
sort: 1
---

# 模板处理

beego 的模板处理引擎采用的是 Go 内置的 `html/template` 包进行处理，而且 beego 的模板处理逻辑是采用了缓存编译方式，也就是所有的模板会在 beego 应用启动的时候全部编译然后缓存在 map 里面。

## 模板目录

beego 中默认的模板目录是 `views`，用户可以把模板文件放到该目录下，beego 会自动在该目录下的所有模板文件进行解析并缓存，开发模式下每次都会重新解析，不做缓存。当然，用户也可以通过如下的方式改变模板的目录（只能指定一个目录为模板目录）：

	beego.ViewsPath = "myviewpath"

## 自动渲染

用户无需手动的调用渲染输出模板，beego 会自动的在调用完相应的 method 方法之后调用 Render 函数，当然如果您的应用是不需要模板输出的，那么可以在配置文件或者在 `main.go` 中设置关闭自动渲染。

配置文件配置如下：

	autorender = false

main.go 文件中设置如下：

	beego.AutoRender = false

## 模板标签

Go 语言的默认模板采用了 `{{` 和 `}}` 作为左右标签，但是我们有时候在开发中可能界面是采用了 AngularJS 开发，他的模板也是这个标签，故而引起了冲突。在 beego 中你可以通过配置文件或者直接设置配置变量修改：

	beego.TemplateLeft = "<<<"
	beego.TemplateRight = ">>>"

## 模板数据

模板中的数据是通过在 Controller 中 `this.Data` 获取的，所以如果你想在模板中获取内容 `{{.Content}}` ,那么你需要在 Controller 中如下设置：

	this.Data["Content"] = "value"

如何使用各种类型的数据渲染：

- 结构体
	
	结构体结构

		type A struct{
			Name string
			Age  int
		}
	
	控制器数据赋值
			
		this.Data["a"]=&A{Name:"astaxie",Age:25}
		
	模板渲染数据如下：
	
		the username is {{.a.Name}} 
		the age is {{.a.Age}}
			
- map
	
	控制器数据赋值
	
		mp["name"]="astaxie"
		mp["nickname"] = "haha"
		this.Data["m"]=mp

	模板渲染数据如下：
	
		the username is {{.m.name}}
		the username is {{.m.nickname}}
		
- slice

	控制器数据赋值
	
		ss :=[]string{"a","b","c"}
		this.Data["s"]=ss
	
	模板渲染数据如下：
	
		{{range $key, $val := .s}}
		{{$key}}
		{{$val}}
	    {{end}}	

## 模板名称

beego 采用了 Go 语言内置的模板引擎，所有模板的语法和 Go 的一模一样，至于如何写模板文件，详细的请参考 [模板教程](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/07.4.md)。

用户通过在 Controller 的对应方法中设置相应的模板名称，beego 会自动的在 viewpath 目录下查询该文件并渲染，例如下面的设置，beego 会在 admin 下面找 add.tpl 文件进行渲染：

	this.TplName = "admin/add.tpl"

我们看到上面的模板后缀名是 tpl，beego 默认情况下支持 tpl 和 html 后缀名的模板文件，如果你的后缀名不是这两种，请进行如下设置：

	beego.AddTemplateExt("你文件的后缀名")

当你设置了自动渲染，然后在你的 Controller 中没有设置任何的 TplName，那么 beego 会自动设置你的模板文件如下：

	c.TplName = strings.ToLower(c.controllerName) + "/" + strings.ToLower(c.actionName) + "." + c.TplExt

也就是你对应的 Controller 名字+请求方法名.模板后缀，也就是如果你的 Controller 名是 `AddController`，请求方法是 `POST`，默认的文件后缀是 `tpl`，那么就会默认请求 `/viewpath/AddController/post.tpl` 文件。

## Layout 设计

beego 支持 layout 设计，例如你在管理系统中，整个管理界面是固定的，只会变化中间的部分，那么你可以通过如下的设置：

	this.Layout = "admin/layout.html"
	this.TplName = "admin/add.tpl" 

在 layout.html 中你必须设置如下的变量：

	{{.LayoutContent}}
 
beego 就会首先解析 TplName 指定的文件，获取内容赋值给 LayoutContent，然后最后渲染 layout.html 文件。

目前采用首先把目录下所有的文件进行缓存，所以用户还可以通过类似这样的方式实现 layout：

	{{template "header.html" .}}
	Logic code
	{{template "footer.html" .}}

>>> 特别注意后面的`.`,这是传递当前参数到子模板

## LayoutSection

对于一个复杂的 `LayoutContent`，其中可能包括有javascript脚本、CSS 引用等，根据惯例，通常 css 会放到 Head 元素中，javascript 脚本需要放到 body 元素的末尾，而其它内容则根据需要放在合适的位置。在 `Layout` 页中仅有一个 `LayoutContent` 是不够的。所以在 `Controller` 中增加了一个 `LayoutSections`属性，可以允许 `Layout` 页中设置多个 `section`，然后每个 `section` 可以分别包含各自的子模板页。

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

逻辑处理如下所示：

```
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

## renderform 使用

定义 struct:

	type User struct {
		Id    int         `form:"-"`
		Name  interface{} `form:"username"`
		Age   int         `form:"age,text,年龄："`
		Sex   string
		Intro string `form:",textarea"`
	}

* StructTag 的定义用的标签用为`form`，和 [ParseForm 方法](../controller/params.md#%E7%9B%B4%E6%8E%A5%E8%A7%A3%E6%9E%90%E5%88%B0-struct) 共用一个标签，标签后面有三个可选参数，用`,`分割。第一个参数为表单中类型的`name`的值，如果为空，则以`struct field name`为值。第二个参数为表单组件的类型，如果为空，则为`text`。表单组件的标签默认为`struct field name`的值，否则为第三个值。
* 如果`form`标签只有一个值，则为表单中类型`name`的值，除了最后一个值可以忽略外，其他位置的必须要有`,`号分割，如：`form:",,姓名："`
* 如果要忽略一个字段，有两种办法，一是：字段名小写开头，二是：`form` 标签的值设置为 `-`
* 现在的代码版本只能实现固定的格式，用br标签实现换行，无法实现css和class等代码的插入。所以，要实现form的高级排版，不能使用renderform的方法，而需要手动处理每一个字段。

controller：

	func (this *AddController) Get() {
	    this.Data["Form"] = &User{}
	    this.TplName = "index.tpl"
	}

Form 的参数必须是一个 struct 的指针。

template:

	<form action="" method="post">
	{{.Form | renderform}}
	</form>

上面的代码生成的表单为：
	
```
	Name: <input name="username" type="text" value="test"></br>
	年龄：<input name="age" type="text" value="0"></br>
	Sex: <input name="Sex" type="text" value=""></br>
	Intro: <input name="Intro" type="textarea" value="">
```	
