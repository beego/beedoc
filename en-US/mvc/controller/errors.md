---
name: Error Handling
sort: 10
---

# Error Handling

In web development, we always need to redirect page and error handling. Beego use `Redirect` to it:

```go
func (this *AddController) Get() {
	this.Redirect("/", 302)
}
```

If you want to stop this request and throw a exception, you can do this in beego's controller:

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

The `this.Abort("401")` won't execute any more and it will show use this page:

![](../../images/401.png)

Beego supports 404, 401, 403, 500, 503 error handling by default. You can also define customer error handling page. For example redefined 404 page:

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

Another cool feature of Go is it supports customized string error handling function, Such as the code below, it registered a database error page:

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

After register this error handling code. you can call `this.Abort("dbError")` in every place in your code to handle the database error.
