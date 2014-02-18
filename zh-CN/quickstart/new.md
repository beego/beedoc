---
name: 新建项目
sort: 1
---

# 创建项目

beego 的项目基本都是通过 `bee` 命令来创建的，所以在创建项目之前确保你已经安装了 bee 工具和 beego。如果你还没有安装，那么请查阅 [beego 的安装](../install) 和 [bee 工具的安装](../install/bee.md)。

现在一切就绪我们就可以开始创建项目了，打开终端，进入 `$GOPATH` 所在的目录：

	➜  src  bee new quickstart
	[INFO] Creating application...
	/gopath/src/quickstart/
	/gopath/src/quickstart/conf/
	/gopath/src/quickstart/controllers/
	/gopath/src/quickstart/models/
	/gopath/src/quickstart/static/
	/gopath/src/quickstart/static/js/
	/gopath/src/quickstart/static/css/
	/gopath/src/quickstart/static/img/
	/gopath/src/quickstart/views/
	/gopath/src/quickstart/conf/app.conf
	/gopath/src/quickstart/controllers/default.go
	/gopath/src/quickstart/views/index.tpl
	/gopath/src/quickstart/main.go
	13-11-26 10:34:10 [SUCC] New application successfully created!
	
通过一个简单的命令就创建了一个 beego 项目。他的目录结构如下所示

	quickstart
	├── conf
	│   └── app.conf
	├── controllers
	│   └── default.go
	├── main.go
	├── models
	├── static
	│   ├── css
	│   ├── img
	│   └── js
	└── views
	    └── index.tpl	

从目录结构中我们也可以看出来这是一个典型的MVC架构的应用，`main.go` 是入口文件。				
## 运行项目

beego 项目创建之后，我们就开始运行项目，首先进入创建的项目，我们使用 `bee run` 来运行该项目，这样就可以做到热编译的效果：

	➜  src  cd quickstart
	➜  quickstart  bee run
	13-11-26 10:43:14 [INFO] Uses 'quickstart' as 'appname'
	13-11-26 10:43:14 [INFO] Initializing watcher...
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart/controllers)
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart/models)
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart)
	13-11-26 10:43:14 [INFO] Start building...

这样我们的应用已经在 `8080` 端口(beego 的默认端口)跑起来了.你是不是觉得很神奇，为什么没有 nginx 和 apache 居然可以自己干这个事情？是的，Go 其实已经做了网络层的东西，beego 只是封装了一下，所以可以做到不需要 nginx 和 apache。让我们打开浏览器看看效果吧：

![](../images/beerun.png)

你内心是否激动了？开发网站如此简单有没有。好了，接下来让我们一层一层的剥离来大概的了解 beego 是怎么运行起来的。