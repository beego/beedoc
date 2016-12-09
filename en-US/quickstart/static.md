---
name: Static files
sort: 6
---

# Handling static files

We talked about how to render a view. Usually there are lots of static files including images, js, css and so on. Our Beego project skeleton has already included folders for these.

```
├── static
│   ├── css
│   ├── img
│   └── js
```

Beego registers the static directory as the static path by default. Registered rule: URL prefix with directory mapping

	StaticDir["/static"] = "static"

You can register multiple static directories. For example if you require two download directories `download1` and `download2` you can set them as:

	beego.SetStaticPath("/down1", "download1")
	beego.SetStaticPath("/down2", "download2")

Visiting the URL `http://localhost/down1/123.txt` will request the file `123.txt` in the `download1` directory.
To remove the default `/static -> static` mapping, you can use `beego.DelStaticPath("/static")`.

# Implementation

To implement this in your Web Application, register your Static directory to your routes.go files

	beego.SetStaticPath("/down1", "download1")

Save it, and now you can access it in your browser.
