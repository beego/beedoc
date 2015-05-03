---
name: View
sort: 5
---

# Creating views

When we created the Controller we used the line `this.TplNames = "index.tpl"` to declare the template to be rendered. By default `beego.Controller` supports `tpl` and `html` extensions. You can call `beego.AddTemplateExt` to add other extensions.

So how can views show the data we need? Beego is using the default `html/template` engine built into Go, so it's Go templates, plain and simple. You can learn how to use Go templates from [*Building Web Applications with Golang*](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/07.4.md).

Let's look at an example:

```
<!DOCTYPE html>

<html>
    <head>
        <title>Beego</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    </head>
    <body>
        <header class="hero-unit" style="background-color:#A9F16C">
            <div class="container">
            <div class="row">
              <div class="hero-text">
                <h1>Welcome to Beego!</h1>
                <p class="description">
                    Beego is a simple & powerful Go web framework which is inspired by tornado and sinatra.
                <br />
                    Official website: <a href="http://{{.Website}}">{{.Website}}</a>
                <br />
                    Contact me: {{.Email}}
                </p>
              </div>
            </div>
            </div>
        </header>
    </body>
</html>
```

We assigned the data into a map type variable `Data` in the controller, which is used as the rendering context. Therefore we can now access and output the data by using the keys `.Website` and `.Email`. 

Next let's talk about [how to serve static files](static.md).
