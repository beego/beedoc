---
name: 路由设置
sort: 2
---

# 项目路由设置

前面我们已经创建了 beego 项目，而且我们也看到了他已经运行起来了，那么是如何运行起来的呢？让我们从入口文件先分析起来吧：

	package main
	
	import (
	        "quickstart/controllers"
	        "github.com/astaxie/beego"
	)
	
	func main() {
	        beego.Router("/", &controllers.MainController{})
	        beego.Run()
	}
	
我们可以看到很简单的两句话，第一句是注册路由 `beego.Router`，第二句是 `beego.Run`，那么这两句话都做了一些什么呢？

1. beego.Router 其实是注册了一个地址，第一个参数是URL(用户请求的地址)，这里我们注册的是 `/`，也就是我们访问的不带任何参数的URL，第二个参数是对应的 Controller，也就是我们即将把请求分发到那个控制器来执行相应的逻辑，我们可以执行类似的方式注册如下路由：

		beego.Router("/user", &controllers.UserController{})	
	这样用户就可以通过访问 `/user` 去执行 `UserController` 的逻辑。这就是我们所谓的路由，更多更复杂的路由规则请查询 [beego 的路由设置](../mvc/controller/router.md)
	
2. beego.Run 执行之后，我们看到的效果好像只是监听服务端口这个过程，但是它内部做了很多事情：
	- 解析配置文件
	
		beego 会自动在 conf 目录下面去解析相应的配置文件 `app.conf`，这样就可以通过配置文件配置一些例如开启的端口，是否开启 session，应用名称等各种信息。
	- 开启 session
	
		会根据上面配置文件的分析之后判断是否开启 session，如果开启的话就初始化全局的 session。		
	- 编译模板
	
		beego 会在启动的时候把 views 目录下的所有模板进行预编译，然后存在 map 里面，这样可以有效的提高模板运行的效率，无需进行多次编译。
	- 启动管理模块
	
		beego 目前做了一个很帅的模块，应用内监控模块，会在 8088 端口做一个内部监听，我们可以通过这个端口查询到 QPS、CPU、内存、GC、goroutine、thread 等各种信息。
	- 监听服务端口
	
		这是最后一步也就是我们看到的访问 8080 看到的网页端口，内部其实调用了 `ListenAndServe`，充分利用了 goroutine 的优势
		
	一旦 run 起来之后，我们的服务就监听在两个端口了，一个服务端口 8080 作为对外服务，另一个 8088 端口实行对内监控。
	
通过这个代码的分析我们了解了 beego 运行起来的过程，以及内部的一些机制。接下来让我们去剥离 Controller 如何来处理逻辑的。	
	 
	
	
