---
name: Static files
sort: 3
---

# Static Files

Go already has the built-in `http.ServeFile` package to serve static files. Beego made a wrapper for it. To register static files use:

	beego.SetStaticPath("/static","public")

- The first parameter is the url path
- The second parameter is the static file directory path. (relative to the application directory)

Beego supports multiple static file directories:

	beego.SetStaticPath("/images","images")
	beego.SetStaticPath("/css","css")
	beego.SetStaticPath("/js","js")

With the above settings, request `/images/login/login.png` will find `application_path/images/login/login.png` and request `/static/img/logo.png` will find `public/img/logo.png` file.

By default Beego will check if the file exists, if not it will return a 404 page.  If the request is for `index.html`, because `http.ServeFile` will redirect and doesn't display this page by default, you can set `beego.BConfig.WebConfig.DirectoryIndex = true` to show `index.html` page. If this is enabled, users can see the file list while visit the directory.
