---
name: flash 数据
sort: 6
---

# flash 数据

这个 flash 与 Adobe/Macromedia Flash 没有任何关系。它主要用于在两个逻辑间传递临时数据，flash 中存放的所有数据会在紧接着的下一个逻辑中调用后清除。一般用于传递提示和错误消息。它适合 [Post/Redirect/Get](http://en.wikipedia.org/wiki/Post/Redirect/Get) 模式。下面看使用的例子：

```go
// 显示设置信息
func (c *MainController) Get() {
	flash:=beego.ReadFromRequest(&c.Controller)
	if n,ok:=flash.Data["notice"];ok{
		//显示设置成功
		c.TplName = "set_success.html"
	}else if n,ok=flash.Data["error"];ok{
		//显示错误
		c.TplName = "set_error.html"
	}else{
		// 不然默认显示设置页面
		c.Data["list"]=GetInfo()
		c.TplName = "setting_list.html"
	}
}

// 处理设置信息
func (c *MainController) Post() {
	flash:=beego.NewFlash()
	setting:=Settings{}
	valid := Validation{}
	c.ParseForm(&setting)
	if b, err := valid.Valid(setting);err!=nil {
		flash.Error("Settings invalid!")
		flash.Store(&c.Controller)
		c.Redirect("/setting",302)
		return
	}else if b!=nil{
		flash.Error("validation err!")
		flash.Store(&c.Controller)
		c.Redirect("/setting",302)
		return
	}	
	saveSetting(setting)
	flash.Notice("Settings saved!")
	flash.Store(&c.Controller)
	c.Redirect("/setting",302)
}
```

上面的代码执行的大概逻辑是这样的：

1. Get 方法执行，因为没有 flash 数据，所以显示设置页面。
2. 用户设置信息之后点击递交，执行 Post，然后初始化一个 flash，通过验证，验证出错或者验证不通过设置 flash 的错误，如果通过了就保存设置，然后设置 flash 成功设置的信息。
3. 设置完成后跳转到 Get 请求。
4. Get 请求获取到了 Flash 信息，然后执行相应的逻辑，如果出错显示出错的页面，如果成功显示成功的页面。

默认情况下 `ReadFromRequest` 函数已经实现了读取的数据赋值给 flash，所以在你的模板里面你可以这样读取数据：

	{{.flash.error}}
	{{.flash.warning}}
	{{.flash.notice}}
	
flash 对象有三个级别的设置：

* Notice 提示信息
* Warning 警告信息
* Error 错误信息
