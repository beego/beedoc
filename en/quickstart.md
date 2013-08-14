# Quickstart

## Installation

Beego contains sample applications to help you learn and use Beego web framework.

You will need a functioning Go 1.1 installation for this to work.

Beego is a "go get" able Go project: `go get github.com/astaxie/beego`

You may also need [Bee](/docs/Reference_BeeTool) tool for developing: `go get github.com/astaxie/bee`

For convenient, you should add `$GOPATH/bin` into $PATH variable.

Want to quick setup an application see if it works?

	$ cd $GOPATH/src
	$ bee new hello
	$ cd hello
	$ bee run hello

These commands help you:

1. Install beego into your $GOPATH.
2. Install Bee tool in your computer.
3. Create a new application called "hello".
4. Start hot compile.

Once it's running, open a browser to [http://localhost:8080/](http://localhost:8080/).

## Simple example

The following example prints string "Hello world" to your browser, it shows how easy to build a web application with beego.

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

Save file as "hello.go", build and run it:

	$ go build hello.go
	$ ./hello

Open address [http://127.0.0.1:8080](http://127.0.0.1:8080) in your browser and you will see "hello world".

What happened in behind above example?

1. We import package `github.com/astaxie/beego`. As we know that Go initialize packages and runs init() function in every package ([more details](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/02.3.md#main-function-and-init-function)), so Beego initializes the BeeApp application at this time.
2. Define controller. We define a struct called `MainController` with a anonymous field `beego.Controller`, so the `MainController` has all methods that `beego.Controller` has.
3. Define RESTful methods. Once we use anonymous combination, `MainController` has already had `Get`, `Post`, `Delete`, `Put` and other methods, these methods will be called when user sends corresponding request, like `Post` method is for requests that are using POST method. Therefore, after we overloaded `Get` method in `MainController`, all GET requests will use `Get` method in `MainController` instead of in `beego.Controller`.
4. Define main function. All applications in Go use main function as entry point as C does.
5. Register routers, it tells beego which controller is responsibility for specific requests. Here we register `/` for `MainController`, so all requests in `/` will be handed to `MainController`. Be aware that the first argument is the path and the second one is pointer of controller that you want to register.
6. Run application in port 8080 as default, press `Ctrl+c` to exit.
