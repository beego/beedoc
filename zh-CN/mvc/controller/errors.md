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
	this.TplNames = "index.tpl"
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
