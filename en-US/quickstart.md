# Getting started

## Installation

Beego contains sample applications to help you learn and use the Beego app framework.

You will need a [Go](https://golang.org) 1.1+ installation for this to work.

You will need to install or upgrade [Beego](http://beego.vip/docs/install/bee.md) and the [Bee](http://beego.vip/docs/install/bee.md) dev tool:

```
go install github.com/beego/beego/v2@latest
go install github.com/beego/bee/v2@latest
```

For convenience, you should add `$GOPATH/bin` to your `$PATH` environment variable. Please make sure you have already set the `$GOPATH` environment variable. 

If you haven't set `$GOPATH` add it to the shell you're using (~/.profile, ~/.zshrc, ~/.cshrc or any other).

For example `~/.zsh`
```
echo 'export GOPATH="$HOME/go"' >> ~/.zsh
```

If you have already set `$GOPATH`
```
echo 'export PATH="$GOPATH/bin:$PATH"' >> ~/.profile # or ~/.zshrc, ~/.cshrc, whatever shell you use
exec $SHELL
```

Want to quickly see how it works? Then just set things up like this:
```
cd $GOPATH/src
bee new hello
cd hello
bee run
```

Windows users：
```
cd %GOPATH%/src
bee new hello
cd hello
bee run
```

These commands help you:

1. Install Beego into your `$GOPATH`.
2. Install the Bee tool in your computer.
3. Create a new application called `hello`.
4. Start hot compile.

Once it's running, open a browser to [http://localhost:8080/](http://localhost:8080/).

## Simple example

The following example prints `Hello world` to your browser, it shows how easy it is to build a web application with beego.

```
package main

import (
	"github.com/beego/beego/v2/server/web"
)

type MainController struct {
	web.Controller
}

func (this *MainController) Get() {
	this.Ctx.WriteString("hello world")
}

func main() {
	web.Router("/", &MainController{})
	web.Run()
}
```

Save file as `hello.go`, build and run it:

	$ go build -o hello hello.go
	$ ./hello

Open [http://127.0.0.1:8080](http://127.0.0.1:8080) in your browser and you will see `hello world`.

What is happening in the scenes of the above example?

1. We import package `github.com/beego/beego/v2/server/web`. As we know, Go initializes packages and runs init() in every package ([more details](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/02.3.md#main-function-and-init-function)), so Beego initializes the `BeeApp` application at this time.
2. Define the controller. We define a struct called `MainController` with an anonymous field `web.Controller`, so the `MainController` has all methods that `web.Controller` has.
3. Define some RESTful methods. Due to the anonymous field above, `MainController` already has `Get`, `Post`, `Delete`, `Put` and other methods, these methods will be called when user sends a corresponding request (e.g. the `Post` method is called to handle requests using POST. Therefore, after we overloaded the `Get` method in `MainController`, all GET requests will use that method in `MainController` instead of in `web.Controller`.
4. Define the main function. All applications in Go use `main` as their entry point like C does.
5. Register routers. This tells Beego which controller is responsible for specific requests. Here we register `MainController` for `/`, so all requests to `/` will be handed by `MainController`. Be aware that the first argument is the path and the second one is pointer to the controller you want to register.
6. Run the application on port 8080 as default, press `Ctrl+c` to exit.

Following are shortcut `.bat` files for Windows users:

Create files  `step1.install-bee.bat` and `step2.new-beego-app.bat` under `%GOPATH%/src`.

`step1.install-bee.bat`:

	set GOPATH=%~dp0..
	go build github.com\beego\bee
	copy bee.exe %GOPATH%\bin\bee.exe
	del bee.exe
	pause

`step2.new-beego-app.bat`:

	@echo Set value of APP same as your app folder
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

Click those two file in order will quick start your Beego tour. And just run `run.bat` in the future.
