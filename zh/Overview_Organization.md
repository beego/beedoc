# 项目组织

Beego 要求自身源码和用户应用程序被安装在合法的 $GOPATH 布局中，换句话说，您必须使用 Go 命令行工具来进行安装（具体参阅 [Go 命令行文档](http://golang.org/cmd/go/) 的 “GOPATH Environment Variable” 部分）。

### 示例布局

下面是一个名为 Sample 的默认 Beego 应用程序项目布局，该布局与典型的 Go 包安装布局类似。

	$GOPATH                         $GOPATH 根目录
	  src                   		$GOPATH/src 目录
	    github.com/astaxie/beego	Beego 源码目录
	      ...
	    sample              App 根目录
		    main.go         Main 包文件
		    controllers     App 控制器目录
		    models          App 模型目录
		    views           模板文件
			conf            配置文件目录
	        	app.conf    主要配置文件
	      	static          静态文件目录
		        css         CSS 文件目录
		        js          Javascript 文件目录
		        img         图片文件目录

这是一个示例的项目结构，您拥有自由修改的权利。比如说，您可以将控制器的目录名 “controllers” 改成 “routers”。