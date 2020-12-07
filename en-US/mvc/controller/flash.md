---
name: Flash messages
sort: 6
---

# Flash Messages

Flash messages are not related to Adobe/Macromedia Flash. They are temporary messages between two logic blocks. All flash messages will be cleared after the very next logic block. They are normally used to send notes and error messages. Their use is suited for the [Post/Redirect/Get](http://en.wikipedia.org/wiki/Post/Redirect/Get) model. For example:

```go
// Display settings message
func (c *MainController) Get() {
    flash := web.ReadFromRequest(&c.Controller)
    if n, ok := flash.Data["notice"]; ok {
        // Display settings successful
        c.TplName = "set_success.html"
    } else if n, ok = flash.Data["error"]; ok {
        // Display error messages
        c.TplName = "set_error.html"
    } else {
        // Display default settings page
        this.Data["list"] = GetInfo()
        c.TplName = "setting_list.html"
    }
}

// Process settings messages
func (c *MainController) Post() {
    flash := web.NewFlash()
    setting := Settings{}
    valid := Validation{}
    c.ParseForm(&setting)
    if b, err := valid.Valid(setting); err != nil {
        flash.Error("Settings invalid!")
        flash.Store(&c.Controller)
        c.Redirect("/setting", 302)
        return
    } else if b != nil {
        flash.Error("validation err!")
        flash.Store(&c.Controller)
        c.Redirect("/setting", 302)
        return
    }
    saveSetting(setting)
    flash.Notice("Settings saved!")
    flash.Store(&c.Controller)
    c.Redirect("/setting", 302)
}
```

The logic of the code above is as follows:

1. Execute GET method. There's no flash data, so display settings page.
2. After submission, execute POST and initialize a flash. If checking failed, set error flash message. If checking passed, save settings and set flash message to successful.
3. Redirect to GET request.
4. GET request receives flash message and executes the related logic. Show error page or success page based on the type of message.

`ReadFromRequest` assigns messages to flash, so you can use it in your template:

	{{.flash.error}}
	{{.flash.warning}}
	{{.flash.success}}
	{{.flash.notice}}

There are 4 different levels of flash messages:

* Notice: Notice message
* Success: Success message
* Warning: Warning message
* Error: Error message
