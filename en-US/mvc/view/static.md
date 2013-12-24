---
name: Static files
sort: 3
---

# Static Files

Go already has buildin `http.ServeFile` to serve static files. Beego made a wrapper for it. To register static files by:

	beego.SetStaticPath("/static","public")

- The first param is url path
- The second param is static file directory path. (relate to applicaion directory)

Beego supports multiple static file directories:

	beego.SetStaticPath("/images","images")
	beego.SetStaticPath("/css","css")
	beego.SetStaticPath("/js","js")

With the above settings, request `/images/login/login.png` will find `application_path/images/login/login.png` and request `/static/img/logo.png` will find `public/img/logo.png` file.

By default beego will check if the file exists, if not return 404 page.  If the request is for `index.html`, because `http.ServeFile` will redirect and don't display this page by default, you can set `beego.DirectoryIndex=true` to show `index.html` page. If this is enabled, use can see the file list while visit the directory.
