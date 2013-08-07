# 模板处理

### 模板目录

Beego 中默认的模板目录是 `views`，用户可以把模板文件放到该目录下，Beego 会自动在该目录下的所有模板文件进行解析并缓存，开发模式下每次都会重新解析，不做缓存。当然，用户也可以通过如下的方式改变模板的目录（只能指定一个目录为模板目录）：

	beego.ViewsPath = "/myviewpath"

### 自动渲染

用户无需手动的调用渲染输出模板，Beego 会自动的在调用完相应的 method 方法之后调用 Render 函数，当然如果您的应用是不需要模板输出的，那么可以在配置文件或者在 main.go 中设置关闭自动渲染。

配置文件配置如下：

	autorender = false

main.go 文件中设置如下：

	beego.AutoRender = false

### 模板数据

模板中的数据是通过在 Controller 中 `this.Data` 获取的，所以如果你想在模板中获取内容 `{{.Content}}` ,那么你需要在 Controller 中如下设置：

	this.Data["Content"] = "value"

### 模板名称

Beego 采用了 Go 语言内置的模板引擎，所有模板的语法和 Go 的一模一样，至于如何写模板文件，详细的请参考 [模板教程](https://github.com/astaxie/build-web-application-with-golang/blob/master/ebook/07.4.md)。

用户通过在 Controller 的对应方法中设置相应的模板名称，Beego 会自动的在 viewpath 目录下查询该文件并渲染，例如下面的设置，Beego 会在 admin 下面找 add.tpl 文件进行渲染：

	this.TplNames = "admin/add.tpl"

我们看到上面的模板后缀名是 tpl，Beego 默认情况下支持 tpl 和 html 后缀名的模板文件，如果你的后缀名不是这两种，请进行如下设置：

	beego.AddTemplateExt("你文件的后缀名")

当你设置了自动渲染，然后在你的 Controller 中没有设置任何的 TplNames，那么 Beego 会自动设置你的模板文件如下：

	c.TplNames = c.ChildName + "/" + c.Ctx.Request.Method + "." + c.TplExt

也就是你对应的 Controller 名字+请求方法名.模板后缀，也就是如果你的 Controller 名是 `AddController`，请求方法是 `POST`，默认的文件后缀是 `tpl`，那么就会默认请求 `/viewpath/AddController/POST.tpl` 文件。

### Layout 设计

Beego 支持 layout 设计，例如你在管理系统中，整个管理界面是固定的，只会变化中间的部分，那么你可以通过如下的设置：

	this.Layout = "admin/layout.html"
	this.TplNames = "admin/add.tpl" 

在 layout.html 中你必须设置如下的变量：

	{{.LayoutContent}}
 
Beego 就会首先解析 TplNames 指定的文件，获取内容赋值给 LayoutContent，然后最后渲染 layout.html 文件。

目前采用首先把目录下所有的文件进行缓存，所以用户还可以通过类似这样的方式实现 layout：

	{{template "header.html"}}
	处理逻辑
	{{template "footer.html"}}

### 模板函数

Beego 支持用户定义模板函数，但是必须在 `beego.Run()` 调用之前，设置如下：

	func hello(in string)(out string){
		out = in + "world"
		return
	}
	
	beego.AddFuncMap("hi",hello)

定义之后你就可以在模板中这样使用了：

	{{.Content | hi}}

目前 Beego 内置的模板函数如下所示：

* markdown

	实现了把 markdown 文本转化为 html 信息，使用方法 {{markdown .Content}}。

* dateformat

	实现了时间的格式化，返回字符串，使用方法 {{dateformat .Time "2006-01-02T15:04:05Z07:00"}}。

* date

	实现了类似 PHP 的 date 函数，可以很方便的根据字符串返回时间，使用方法 {{date .T "Y-m-d H:i:s"}}。

* compare

	实现了比较两个对象的比较，如果相同返回 true，否者 false，使用方法 {{compare .A .B}}。

* substr

	实现了字符串的截取，支持中文截取的完美截取，使用方法 {{substr .Str 0 30}}。

* html2str

	实现了把 html 转化为字符串，剔除一些 script、css 之类的元素，返回纯文本信息，使用方法 {{html2str .Htmlinfo}}。

* str2html

	实现了把相应的字符串当作 HTML 来输出，不转义，使用方法 {{str2html .Strhtml}}。

* htmlquote

	实现了基本的 html 字符转义，使用方法 {{htmlquote .quote}}。

* htmlunquote

	实现了基本的反转移字符，使用方法 {{htmlunquote .unquote}}。
