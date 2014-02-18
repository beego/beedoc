---
name: Controller
sort: 3
---

# Controller logic

We learned how we distribute the users' processes to controllers in previous section. We will learn how to write a controller in this section. Let's start with some code:

```
package controllers

import (
        "github.com/astaxie/beego"
)

type MainController struct {
        beego.Controller
}

func (this *MainController) Get() {
        this.Data["Website"] = "beego.me"
        this.Data["Email"] = "astaxie@gmail.com"
        this.TplNames = "index.tpl"
}
```

At the beginning it created the `MainController` and it contains `beego.Controller`. This is the way how Go embeds. It means `MainController` automatically acquired all the methods of `beego.Controller`.

`beego.Controller` has lots of methods such as `Init`, `Prepare`, `Post`, `Get`, `Delete`, `Head`. We can implement these functions by overwriting them.In this case we overwrited `GET` method.

We talked about Beego is a RESTFul framework so our requests will run the related `req.Method` method by default. For example, if the browser send a `GET` request, it will execute `GET` method in `MainController`. The `GET` method and the logic we defined above will be executed.

The logic in our `GET` method is just output some data. We can get our data by many ways and assign it into `this.Data` which is a map storing our data. We can assign any type of data here we just assigned two strings.

The last thing is rendering the template. `this.TplNames` is the template will be rendered here it's `index.tpl`. If you don't set the template, it will find `controller/method_name.tpl` by default. For example, in this case it will try to find `maincontroller/get.tpl`.

Beego will call `Render` function (it's implemented in `beego.Controller`) automatically if use set up the template so user don't need to render it manually.

We discussed about Controller. Next we will talk about how can we write models.
