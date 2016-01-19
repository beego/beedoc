---
name: 错误处理
sort: 10
---

# 错误处理

我们在做 Web 开发的时候，经常会遇到页面调整和错误处理，beego 这这方面也进行了考虑，通过 `Redirect` 方法来进行跳转：

```go
func (this *AddController) Get() {
	this.Redirect("/", 302)
}
```

如何中止此次请求并抛出异常，beego 可以在控制器中这操作：

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
	this.TplName = "index.tpl"
}
```

这样 `this.Abort("401")` 之后的代码不会再执行，而且会默认显示给用户如下页面：

![](../../images/401.png)

beego 框架默认支持 404、401、403、500、503 这几种错误的处理。用户可以自定义相应的错误处理，例如下面重新定义 404 页面：

```go
func page_not_found(rw http.ResponseWriter, r *http.Request){
	t,_:= template.New("404.html").ParseFiles(beego.ViewsPath+"/404.html")
	data :=make(map[string]interface{})
	data["content"] = "page not found"
	t.Execute(rw, data)
}

func main() {
	beego.Errorhandler("404",page_not_found)
	beego.Router("/", &controllers.MainController{})
	beego.Run()
}
```

我们可以通过自定义错误页面 `404.html` 来处理 404 错误。

beego 更加人性化的还有一个设计就是支持用户自定义字符串错误类型处理函数，例如下面的代码，用户注册了一个数据库出错的处理页面：

```go
func dbError(rw http.ResponseWriter, r *http.Request){
	t,_:= template.New("dberror.html").ParseFiles(beego.ViewsPath+"/dberror.html")
	data :=make(map[string]interface{})
	data["content"] = "database is now down"
	t.Execute(rw, data)
}

func main() {
	beego.Errorhandler("dbError",dbError)
	beego.Router("/", &controllers.MainController{})
	beego.Run()
}
```

一旦在入口注册该错误处理代码，那么你可以在任何你的逻辑中遇到数据库错误调用 `this.Abort("dbError")` 来进行异常页面处理。

# Controller定义Error
从1.4.3版本开始，支持Controller方式定义Error错误处理函数，这样就可以充分利用系统自带的模板处理，以及context等方法。

```
package controllers

import (
	"github.com/astaxie/beego"
)

type ErrorController struct {
	beego.Controller
}

func (c *ErrorController) Error404() {
	c.Data["content"] = "page not found"
	c.TplName = "404.tpl"
}

func (c *ErrorController) Error501() {
	c.Data["content"] = "server error"
	c.TplName = "501.tpl"
}


func (c *ErrorController) ErrorDb() {
	c.Data["content"] = "database is now down"
	c.TplName = "dberror.tpl"
}
```
通过上面的例子我们可以看到，所有的函数都是有一定规律的，都是`Error`开头，后面的名字就是我们调用`Abort`的名字，例如`Error404`函数其实调用对应的就是`Abort("404")`


我们就只要在`beego.Run`之前采用`beego.ErrorController`注册这个错误处理函数就可以了

```
package main

import (
	_ "btest/routers"
	"btest/controllers"

	"github.com/astaxie/beego"
)

func main() {
	beego.ErrorController(&controllers.ErrorController{})
	beego.Run()
}

```
