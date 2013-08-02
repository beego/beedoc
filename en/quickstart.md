# Installation

Beego contains sample applications to help you learn and use Beego web framework.

You will need a functioning Go 1.1 installation for this to work.

Beego is a "go get" able Go project: `go get github.com/astaxie/beego`

You may also need [Bee](/docs/Reference_BeeTool) tool for developing: `go get github.com/astaxie/bee`

Want to quick setup an application see if it works?

	$ cd $GOPATH/src
	$ cp $GOPATH/bin/bee bee
	$ ./bee new hello
	$ cd hello
	$ mv ../bee bee
	$ ./bee run hello

These commands help you:

1. Install Beego into your $GOPATH.
2. Install Bee tool in your computer.
3. Create a new application called "hello".
4. Start hot compile.

Once it's running, open a browser to [http://localhost:8080/](http://localhost:8080/).

# Simple example

The following example prints string "Hello world" to your browser, it shows how easy to build a web application with Beego.

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