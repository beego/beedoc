---
name: Controller
sort: 3
---

# Controller logic

The previous section covered user requests to controllers. This section will explain how to write a controller. Let's start with some code:

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
        this.TplNames = "index.tpl" // version 1.6 use this.TplName = "index.tpl"
}
```

The following is a breakdown of the different sections of this code.

## How Beego dispatches requests

The `MainController` is the first thing created. It contains an anonymous struct field of type `beego.Controller`. This is called struct embedding and is the way that Go mimics inheritance. Because of this `MainController` automatically acquires all the methods of `beego.Controller`.

`beego.Controller` has several functions such as `Init`, `Prepare`, `Post`, `Get`, `Delete` and `Head`. These functions can be overwritten by implementing them. In this example the `Get` method was overwritten.

We talked about the fact that Beego is a RESTful framework so our requests will run the related `req.Method` method by default. For example, if the browser sends a `GET` request, it will execute the `Get` method in `MainController`. Therefore the `Get` method and the logic we defined above will be executed.

## The `Get` method

The logic of the `Get` method only outputs data. This data will be stored in `this.Data`, a `map[interface{}]interface{}`.  Any type of data can be assigned here. In this case only two strings are assigned.

Finally the template will be rendered. `this.TplNames` (v1.6 uses `this.TplName`) specifies the template which will be rendered. In this case it is `index.tpl`.  If a template is not set it will default to `controller/method_name.tpl`. For example, in this case it would try to find `maincontroller/get.tpl`.

There is no need to render manually.  Beego will call the `Render` function (which is implemented in `beego.Controller`) automatically if it is set up in the template.

Check the controller section in the [MVC Introduction](../mvc/) to learn more about these functions. [The next section](model.md) will describe model writing.
