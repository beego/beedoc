---
name: Flash messages
sort: 6
---

# Flash Messages

This flash it not related to Adobe/Macromedia Flash at all. It's the temporary messages between two logic block. All the flash messages will be clear after the very next logic block. Usually it is used to transform notes and error messages. It is good for [Post/Redirect/Get](http://en.wikipedia.org/wiki/Post/Redirect/Get) model. For example:

```go
// Display settings message
func (c *MainController) Get() {
	flash:=beego.ReadFromRequest(&c.Controller)
	if n,ok:=flash.Data["notice"];ok{
    // Display settings succeful
		c.TplNames = "set_success.html"
	}else if n,ok=flash.Data["error"];ok{
    // Display error messages
		c.TplNames = "set_error.html"
	}else{
    // Display default settings page
		this.Data["list"]=GetInfo()
		c.TplNames = "setting_list.html"
	}
}

// Process settings messages
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

The logic of code above:

1. Executing GET method. There's no flash data, so display settings page.
2. After submittion execute POST and initialize a flash. If checking faild set error flash message. If pass, seve settings and set success flash message.
3. After setting redirect to GET request.
4. GET request receive flash message and execute the related logic. Show error page or success page based on the type of message.

`ReadFromRequest` already implemented assigning message to flash by default, so you can use it in your template:

	{{.flash.error}}
	{{.flash.warning}}
	{{.flash.notice}}
	
There are 3 different levels' flash messages:

* Notice: Notice message
* Warning: Warning message
* Error: Error message
