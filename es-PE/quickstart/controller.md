---
name: Controller
sort: 3
---

# Controller logic

We learned how we distribute the users' processes to controllers in the previous section. In this section we will learn how to write a controller. Let's start with some code:

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

At the beginning we create the `MainController` and it contains an anonymous struct field of type `beego.Controller`. This is called struct embedding and is the way how Go mimics inheritance. This means `MainController` automatically acquires all the methods of `beego.Controller`.

`beego.Controller` has lots of methods such as `Init`, `Prepare`, `Post`, `Get`, `Delete` and `Head`. We can overwrite these functions by implementing them. In this case we overwrote `GET` method.

We talked about the fact that Beego is a RESTFul framework so our requests will run the related `req.Method` method by default. For example, if the browser sends a `GET` request, it will execute the `Get` method in `MainController`. Therefore the `Get` method and the logic we defined above will be executed.

The logic in our `Get` method just outputs some data. We can get our data by many ways and store it in `this.Data` which is a map[string]interface{}. We can assign any type of data here. In this case we just assigned two strings.

The last thing to be done is rendering the template. `this.TplNames` specifies the template which will be rendered: Here it's `index.tpl`. If you don't set the template, it will default to `controller/method_name.tpl`. For example, in this case it would try to find `maincontroller/get.tpl`.

Beego will call the `Render` function (which is implemented in `beego.Controller`) automatically if you set up the template so you don't need to render it manually.

This was only a brief introduction. Check out the controller section in the MVC introduction to learn more. Next we will talk about how to write models.
