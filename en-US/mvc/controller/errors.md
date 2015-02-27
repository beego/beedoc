---
name: Error Handling
sort: 10
---

# Error Handling

In web development, we need to be able to redirect pages and handle errors. Beego uses `Redirect` for it:

```go
func (this *AddController) Get() {
	this.Redirect("/", 302)
}
```

If you want to stop this request and throw exception, you can do this in beego's controller:

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

The `this.Abort("401")` won't execute any more and it will show this page:

![](../../images/401.png)

Beego supports 404, 401, 403, 500, 503 error handling by default. You can also define custom error handling page. For example redefined 404 page:

```go
func page_not_found(rw http.ResponseWriter, r *http.Request){
	t,_:= template.ParseFiles(beego.ViewsPath+"/404.html")
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

We can define our own `404.html` to handle 404 error.

Another cool feature of Go its support of customized string error handling function, Such as the code below which registers a database error page:

```go
func dbError(rw http.ResponseWriter, r *http.Request){
	t,_:= template.ParseFiles(beego.ViewsPath+"/dberror.html")
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

After registering this error handling code. you can call `this.Abort("dbError")` in every place in your code to handle the database error.

# Controller define Error
from beego 1.4.3，support Controller define Error handle，so we can use the beego.Controller and the template reander, useful context function。

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
	c.TplNames = "404.tpl"
}

func (c *ErrorController) Error501() {
	c.Data["content"] = "server error"
	c.TplNames = "501.tpl"
}


func (c *ErrorController) ErrorDb() {
	c.Data["content"] = "database is now down"
	c.TplNames = "dberror.tpl"
}
```
from the example we can know that all the error handle function hasprefix `Error`，the other string is the name of `Abort`，like `Error404` match `Abort("404")`


we should use `beego.ErrorController` to register the error controller before `beego.Run` 

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