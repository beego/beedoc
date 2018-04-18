---
name: 静态文件处理
sort: 6
---

# 静态文件处理

前面我们介绍了如何输出静态页面，但是我们的网页往往包含了很多的静态文件，包括图片、JS、CSS 等，刚才创建的应用里面就创建了如下目录：

```
├── static
	│   ├── css
	│   ├── img
	│   └── js
```

beego 默认注册了 static 目录为静态处理的目录，注册样式：URL 前缀和映射的目录（在/main.go文件中beego.Run()之前加入）：

	StaticDir["/static"] = "static"

用户可以设置多个静态文件处理目录，例如你有多个文件下载目录 download1、download2，你可以这样映射（在 /main.go 文件中 beego.Run() 之前加入）：

	beego.SetStaticPath("/down1", "download1")
	beego.SetStaticPath("/down2", "download2")

这样用户访问 URL `http://localhost:8080/static/down1/123.txt` 则会请求 download1 目录下的 123.txt 文件。
