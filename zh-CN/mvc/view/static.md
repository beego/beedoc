---
name: 静态文件处理
sort: 3
---

# 静态文件

Go 语言内部其实已经提供了 `http.ServeFile`，通过这个函数可以实现静态文件的服务。beego 针对这个功能进行了一层封装，通过下面的方式进行静态文件注册：

	beego.SetStaticPath("/static","public")

- 第一个参数是路径，url 路径信息
- 第二个参数是静态文件目录（相对应用所在的目录）

beego 支持多个目录的静态文件注册，用户可以注册如下的静态文件目录：

	beego.SetStaticPath("/images","images")
	beego.SetStaticPath("/css","css")
	beego.SetStaticPath("/js","js")

设置了如上的静态目录之后，用户访问 `/images/login/login.png`，那么就会访问应用对应的目录下面的 `images/login/login.png` 文件。如果是访问 `/static/img/logo.png`，那么就访问 `public/img/logo.png`文件。

默认情况下 beego 会判断目录下文件是否存在，不存在直接返回 404 页面，如果请求的是 `index.html`，那么由于 `http.ServeFile` 默认是会跳转的，不提供该页面的显示。因此 beego 可以设置 `beego.BConfig.WebConfig.DirectoryIndex=true` 这样来使得显示 `index.html` 页面。而且开启该功能之后，用户访问目录就会显示该目录下所有的文件列表。
