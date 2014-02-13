---
name: Bee 工具的使用
sort: 2
---

# bee 工具简介

bee 工具是一个为了协助快速开发 beego 项目而创建的项目，您可以通过 bee 快速创建项目、实现热编译、开发测试以及开发完之后打包发布的一整套从创建、开发到部署的方案。

## bee 工具的安装

您可以通过如下的方式安装 bee 工具：

	go get github.com/beego/bee
	
安装完之后，`bee`可执行文件默认存放在`GOPATH/bin`里面，所以您需要把`GOPATH/bin`添加到您的环境变量中，才可以进行下一步。

## bee 工具命令详解

我们在命令行输入 `bee`，可以看到如下的信息：

```
Bee is a tool for managing beego framework.

Usage:

	bee command [arguments]

The commands are:

    new         create an application base on beego framework
    run         run the app which can hot compile
    pack        compress an beego project
    api         create an api application base on beego framework
    router      auto-generate routers for the app controllers
    test        test the app
    bale        packs non-Go files to Go source files	
```	

### new 命令

`new` 命令是新建一个 Web 项目，我们在命令行下执行 `bee new <项目名>` 就可以创建一个新的项目。但是注意该命令必须在 `$GOPATH` 下执行。最后会在 `$GOPATH` 相应目录下生成如下目录结构的项目：

```
bee new myproject
[INFO] Creating application...
/gopath/src/myproject/
/gopath/src/myproject/conf/
/gopath/src/myproject/controllers/
/gopath/src/myproject/models/
/gopath/src/myproject/static/
/gopath/src/myproject/static/js/
/gopath/src/myproject/static/css/
/gopath/src/myproject/static/img/
/gopath/src/myproject/views/
/gopath/src/myproject/conf/app.conf
/gopath/src/myproject/controllers/default.go
/gopath/src/myproject/views/index.tpl
/gopath/src/myproject/main.go
13-11-25 09:50:39 [SUCC] New application successfully created!
```

```
myproject
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

8 directories, 4 files
```

### run 命令

我们在开发 Go 项目的时候最大的问题是经常需要自己手动去编译再运行，`bee run` 命令是监控 beego 的项目，通过 inotify 监控文件系统。这样我们在开发过程中就可以实时的看到项目修改之后的效果：

```
bee run
13-11-25 09:53:04 [INFO] Uses 'myproject' as 'appname'
13-11-25 09:53:04 [INFO] Initializing watcher...
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/controllers)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/models)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject)
13-11-25 09:53:04 [INFO] Start building...
13-11-25 09:53:16 [SUCC] Build was successful
13-11-25 09:53:16 [INFO] Restarting myproject ...
13-11-25 09:53:16 [INFO] ./myproject is running...
```
我们打开浏览器就可以看到效果 `http://localhost:8080/`:

![](../images/beerun.png)

如果我们修改了 `Controller` 下面的 `default.go` 文件，我们就可以看到命令行输出：

```
13-11-25 10:11:20 [EVEN] "/gopath/src/myproject/controllers/default.go": DELETE|MODIFY
13-11-25 10:11:20 [INFO] Start building...
13-11-25 10:11:20 [SKIP] "/gopath/src/myproject/controllers/default.go": CREATE
13-11-25 10:11:23 [SKIP] "/gopath/src/myproject/controllers/default.go": MODIFY
13-11-25 10:11:23 [SUCC] Build was successful
13-11-25 10:11:23 [INFO] Restarting myproject ...
13-11-25 10:11:23 [INFO] ./myproject is running...
```

刷新浏览器我们看到新的修改内容已经输出。

### api 命令

上面的 `new` 命令是用来新建 Web 项目，不过不乏使用 beego 来开发 API 应用用户，提供对外的 API。所以这个 `api` 命令就是用来创建 API 应用的，执行命令之后如下所示：

```
bee api apiproject
create app folder: /gopath/src/apiproject
create conf: /gopath/src/apiproject/conf
create controllers: /gopath/src/apiproject/controllers
create models: /gopath/src/apiproject/models
create tests: /gopath/src/apiproject/tests
create conf app.conf: /gopath/src/apiproject/conf/app.conf
create controllers default.go: /gopath/src/apiproject/controllers/default.go
create tests default.go: /gopath/src/apiproject/tests/default_test.go
create models object.go: /gopath/src/apiproject/models/object.go
create main.go: /gopath/src/apiproject/main.go
```
这个项目的目录结构如下：

```
apiproject
├── conf
│   └── app.conf
├── controllers
│   └── default.go
├── main.go
├── models
│   └── object.go
└── tests
    └── default_test.go
```

从上面的目录我们可以看到和 Web 项目相比，少了 static 和 views 目录，多了一个 test 模块，用来做单元测试的。

### test 命令

这是基于 `go test` 进行封装的一个命令，执行 beego 项目 test 目录下的测试用例：

```
bee test apiproject
13-11-25 10:46:57 [INFO] Initializing watcher...
13-11-25 10:46:57 [TRAC] Directory(/gopath/src/apiproject/controllers)
13-11-25 10:46:57 [TRAC] Directory(/gopath/src/apiproject/models)
13-11-25 10:46:57 [TRAC] Directory(/gopath/src/apiproject)
13-11-25 10:46:57 [INFO] Start building...
13-11-25 10:46:58 [SUCC] Build was successful
13-11-25 10:46:58 [INFO] Restarting apiproject ...
13-11-25 10:46:58 [INFO] ./apiproject is running...
13-11-25 10:46:58 [INFO] Start testing...
13-11-25 10:46:59 [TRAC] ============== Test Begin ===================
PASS
ok  	apiproject/tests	0.100s
13-11-25 10:47:00 [TRAC] ============== Test End ===================
13-11-25 10:47:00 [SUCC] Test finish
```

### pack 命令

`pack` 目录用来发布应用的时候打包，会把项目打包成 zip 包，这样我们部署的时候直接打包之后的项目上传，解压就可以部署了：

```
bee pack
app path: /gopath/src/apiproject
GOOS darwin GOARCH amd64
build apiproject
build success
exclude prefix:
exclude suffix: .go:.DS_Store:.tmp
file write to `/gopath/src/apiproject/apiproject.tar.gz`
```

我们可以看到目录下有如下的压缩文件：

```
rwxr-xr-x  1 astaxie  staff  8995376 11 25 22:46 apiproject
-rw-r--r--  1 astaxie  staff  2240288 11 25 22:58 apiproject.tar.gz
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 conf
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 controllers
-rw-r--r--  1 astaxie  staff      509 11 25 22:31 main.go
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 models
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 tests
```

### router 命令

这个命令目前还没有作用，将来会用来分析 controller 的方法，自动生成路由

### bale 命令

这个命令目前仅限内部使用，具体实现方案未完善，主要用来压缩所有的静态文件变成一个变量申明文件，全部编译到二进制文件里面，用户发布的时候携带静态文件，包括 js、css、img 和 views。最后在启动运行时进行非覆盖式的自解压。

## bee 工具配置文件

您可能已经注意到，在 bee 工具的源码目录下有一个 `bee.json` 文件，这个文件是针对 bee 工具的一些行为进行配置。该功能还未完全开发完成，不过其中的一些选项已经可以使用：

- `"version": 0`：配置文件版本，用于对比是否发生不兼容的配置格式版本。
- `"go_install": false`：如果您的包均使用完整的导入路径（例如：`github.com/user/repo/subpkg`）,则可以启用该选项来进行 `go install` 操作，加快构建操作。
- `"watch_ext": []`：用于监控其它类型的文件（默认只监控后缀为 `.go` 的文件）。
- `"dir_structure":{}`：如果您的目录名与默认的 MVC 架构的不同，则可以使用该选项进行修改。
- `"cmd_args": []`：如果您需要在每次启动时加入启动参数，则可以使用该选项。
- `"envs": []`：如果您需要在每次启动时设置临时环境变量参数，则可以使用该选项。