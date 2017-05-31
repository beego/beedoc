# 快速入门

## 安装

beego 包含一些示例应用程序以帮您学习并使用 beego 应用框架。

您需要安装 Go 1.1+ 以确保所有功能的正常使用。

你需要安装或者升级 Beego 和 [Bee](http://beego.me/docs/install/bee.md) 的开发工具:

	$ go get -u github.com/astaxie/beego
	$ go get -u github.com/beego/bee

为了更加方便的操作，请将 `$GOPATH/bin` 加入到你的 `$PATH` 变量中。请确保在此之前您已经添加了 `$GOPATH` 变量。

	# 如果您还没添加 $GOPATH 变量
	$ echo 'export GOPATH="$HOME/go"' >> ~/.profile # 或者 ~/.zshrc, ~/.cshrc, 您所使用的sh对应的配置文件
	
	# 如果您已经添加了 $GOPATH 变量
	$ echo 'export PATH="$GOPATH/bin:$PATH"' >> ~/.profile # 或者 ~/.zshrc, ~/.cshrc, 您所使用的sh对应的配置文件
	$ exec $SHELL

想要快速建立一个应用来检测安装？

	$ cd $GOPATH/src
	$ bee new hello
	$ cd hello
	$ bee run hello

Windows 平台下输入：

    >cd %GOPATH%/src
    >bee new hello
    >cd hello
    >bee run hello

这些指令帮助您：

1. 安装 beego 到您的 $GOPATH 中。
2. 在您的计算机上安装 Bee 工具。
3. 创建一个名为 “hello” 的应用程序。
4. 启动热编译。

一旦程序开始运行，您就可以在浏览器中打开 [http://localhost:8080/](http://localhost:8080/) 进行访问。

## 简单示例

下面这个示例程序将会在浏览器中打印 “Hello world”，以此说明使用 beego 构建 Web 应用程序是多么的简单！

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

把上面的代码保存为 hello.go，然后通过命令行进行编译并执行：

	$ go build -o hello hello.go
	$ ./hello

这个时候你可以打开你的浏览器，通过这个地址浏览 [http://127.0.0.1:8080](http://127.0.0.1:8080) 返回 “hello world”。

那么上面的代码到底做了些什么呢？

1. 首先我们导入了包 `github.com/astaxie/beego`。我们知道 Go 语言里面被导入的包会按照深度优先的顺序去执行导入包的初始化（变量和 init 函数，[更多详情](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.3.md#main函数和init函数)），beego 包中会初始化一个 BeeAPP 的应用和一些参数。
2. 定义 Controller，这里我们定义了一个 struct 为 `MainController`，充分利用了 Go 语言的组合的概念，匿名包含了 `beego.Controller`，这样我们的 `MainController` 就拥有了 `beego.Controller` 的所有方法。
3. 定义 RESTful 方法，通过匿名组合之后，其实目前的 `MainController` 已经拥有了 `Get`、`Post`、`Delete`、`Put` 等方法，这些方法是分别用来对应用户请求的 Method 函数，如果用户发起的是 POST 请求，那么就执行 `Post` 函数。所以这里我们定义了 `MainController` 的 `Get` 方法用来重写继承的 `Get` 函数，这样当用户发起 GET 请求的时候就会执行该函数。
4. 定义 main 函数，所有的 Go 应用程序和 C 语言一样都是 main 函数作为入口，所以我们这里定义了我们应用的入口。
5. Router 注册路由，路由就是告诉 beego，当用户来请求的时候，该如何去调用相应的 Controller，这里我们注册了请求 `/` 的时候，请求到 `MainController`。这里我们需要知道，Router 函数的两个参数函数，第一个是路径，第二个是 Controller 的指针。
6. Run 应用，最后一步就是把在步骤 1 中初始化的 BeeApp 开启起来，其实就是内部监听了 8080 端口：Go 默认情况会监听你本机所有的 IP 上面的 8080 端口。

停止服务的话，请按 `Ctrl+c`。


下面为 windows 下的快捷操作批处理文件：
在 `%GOPATH%/src` 目录下分别创建文件 `step1.install-bee.bat` 和 `step2.new-beego-app.bat`。

`step1.install-bee.bat` 文件内容：

	set GOPATH=%~dp0..
	go build github.com\beego\bee
	copy bee.exe %GOPATH%\bin\bee.exe
	del bee.exe
	pause

`step2.new-beego-app.bat` 文件内容：

	@echo 设置 App 的值为您的应用文件夹名称
	set APP=coscms.com
	set GOPATH=%~dp0..
	set BEE=%GOPATH%\bin\bee
	%BEE% new %APP%
	cd %APP%
	echo %BEE% run %APP%.exe > run.bat
	echo pause >> run.bat
	start run.bat
	pause
	start http://127.0.0.1:8080

依次点击上面创建的两个文件即可快速开启 beego 之旅。
以后只需要到您的应用目录下点击 `run.bat` 即可。
