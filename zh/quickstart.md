# 安装

Beego 包含一些示例应用程序以帮您学习并使用 Beego Web 框架。

您需要安装 Go 1.1 以确保所有功能的正常使用。

Beego 是可以通过 “go get” 安装的 Go 项目：`go get github.com/astaxie/beego`

您或许希望安装 [Bee](/docs/Reference_BeeTool) 工具以协助您开发：`go get github.com/astaxie/bee`

想要快速建立一个应用来检测安装？

	$ cd $GOPATH/src
	$ cp $GOPATH/bin/bee bee
	$ ./bee new hello
	$ cd hello
	$ mv ../bee bee
	$ ./bee run hello

这些指令帮助您：

1. 安装 Beego 到您的 $GOPATH 中。
2. 在您的计算机上安装 Bee 工具。
3. 创建一个名为 “hello” 的应用程序。
4. 启动热编译。

一旦程序开始运行，您就可以在浏览器中打开 [http://localhost:8080/](http://localhost:8080/) 进行访问。

# 简单示例

下面这个示例程序将会在浏览器中打印 “Hello world”，以此说明使用 Beego 构建 Web 应用程序是多么的简单！

	package main
	
	import (
		"github.com/astaxie/beego"
	)
	
	type MainController struct {
		beego.Controller
	}
	
	func (this *MainController) Get() {
		this.Ctx.WriteString("hello world")
	}
	
	func main() {
		beego.Router("/", &MainController{})
		beego.Run()
	}