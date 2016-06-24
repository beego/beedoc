---
name: Bee 工具的使用
sort: 2
---

# bee 工具简介

bee 工具是一个为了协助快速开发 beego 项目而创建的项目，通过bee您可以很容易的进行beego项目的创建、热编译、开发、测试、和部署。

## bee 工具的安装

您可以通过如下的方式安装 bee 工具：

	go get github.com/beego/bee
	
安装完之后，`bee`可执行文件默认存放在`$GOPATH/bin`里面，所以您需要把`$GOPATH/bin`添加到您的环境变量中，才可以进行下一步。

>>> 如何添加环境变量，请自行搜索
>>> 如果你本机设置了`GOBIN`，那么上面的命令就会安装到`GOBIN`下，请添加GOBIN到你的环境变量中

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
    bale        packs non-Go files to Go source files
    version     show the bee & beego version
    generate    source code generator
    migrate     run database migrations
```	

### new 命令

`new` 命令是新建一个 Web 项目，我们在命令行下执行 `bee new <项目名>` 就可以创建一个新的项目。但是注意该命令必须在 `$GOPATH/src` 下执行。最后会在 `$GOPATH/src` 相应目录下生成如下目录结构的项目：

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
│   └── app.conf
├── controllers
│   └── default.go
├── main.go
├── models
├── routers
│   └── router.go
├── static
│   ├── css
│   ├── img
│   └── js
├── tests
│   └── default_test.go
└── views
    └── index.tpl

8 directories, 4 files
```

### api 命令

上面的 `new` 命令是用来新建 Web 项目，不过很多用户使用 beego 来开发 API 应用。所以这个 `api` 命令就是用来创建 API 应用的，执行命令之后如下所示：

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
│   └── object.go
│   └── user.go
├── docs
│   └── doc.go
├── main.go
├── models
│   └── object.go
│   └── user.go
├── routers
│   └── router.go
└── tests
    └── default_test.go
```

从上面的目录我们可以看到和 Web 项目相比，少了 static 和 views 目录，多了一个 test 模块，用来做单元测试的。

同时，该命令还支持一些自定义参数自动连接数据库创建相关model和controller:
bee api [appname] [-tables=""] [-driver=mysql] [-conn=root:@tcp(127.0.0.1:3306)/test]
如果conn参数为空则创建一个示例项目，否则将基于链接信息链接数据库创建项目。

### run 命令

我们在开发 Go 项目的时候最大的问题是经常需要自己手动去编译再运行，`bee run` 命令是监控 beego 的项目，通过 [fsnotify](https://github.com/howeyc/fsnotify) 监控文件系统。这样我们在开发过程中就可以实时的看到项目修改之后的效果：

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

### pack 命令

`pack` 目录用来发布应用的时候打包，会把项目打包成 zip 包，这样我们部署的时候直接把打包之后的项目上传，解压就可以部署了：

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

### bale 命令

这个命令目前仅限内部使用，具体实现方案未完善，主要用来压缩所有的静态文件变成一个变量申明文件，全部编译到二进制文件里面，用户发布的时候携带静态文件，包括 js、css、img 和 views。最后在启动运行时进行非覆盖式的自解压。

### version 命令

这个命令是动态获取bee、beego和Go的版本，这样一旦用户出现错误，可以通过该命令来查看当前的版本

```
$ bee version
bee   :1.2.2
beego :1.4.2
Go    :go version go1.3.3 darwin/amd64
```

### generate 命令
这个命令是用来自动化的生成代码的，包含了从数据库一键生成model，还包含了scaffold的，通过这个命令，让大家开发代码不再慢

```
bee generate scaffold [scaffoldname] [-fields=""] [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    The generate scaffold command will do a number of things for you.
    -fields: a list of table fields. Format: field:type, ...
    -driver: [mysql | postgres | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test
    example: bee generate scaffold post -fields="title:string,body:text"

bee generate model [modelname] [-fields=""]
    generate RESTful model based on fields
    -fields: a list of table fields. Format: field:type, ...

bee generate controller [controllerfile]
    generate RESTful controllers

bee generate view [viewpath]
    generate CRUD view in viewpath

bee generate migration [migrationfile] [-fields=""]
    generate migration file for making database schema update
    -fields: a list of table fields. Format: field:type, ...

bee generate docs
    generate swagger doc file

bee generate test [routerfile]
    generate testcase

bee generate appcode [-tables=""] [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"] [-level=3]
    generate appcode based on an existing database
    -tables: a list of table names separated by ',', default is empty, indicating all tables
    -driver: [mysql | postgres | sqlite], the default is mysql
    -conn:   the connection string used by the driver.
             default for mysql:    root:@tcp(127.0.0.1:3306)/test
             default for postgres: postgres://postgres:postgres@127.0.0.1:5432/postgres
    -level:  [1 | 2 | 3], 1 = models; 2 = models,controllers; 3 = models,controllers,router
```

### migrate 命令
这个命令是应用的数据库迁移命令，主要是用来每次应用升级，降级的SQL管理。

```
bee migrate [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    run all outstanding migrations
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate rollback [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback the last migration operation
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate reset [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback all migrations
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test

bee migrate refresh [-driver=mysql] [-conn="root:@tcp(127.0.0.1:3306)/test"]
    rollback all migrations and run them all again
    -driver: [mysql | postgresql | sqlite], the default is mysql
    -conn:   the connection string used by the driver, the default is root:@tcp(127.0.0.1:3306)/test
```

## bee 工具配置文件

您可能已经注意到，在 bee 工具的源码目录下有一个 `bee.json` 文件，这个文件是针对 bee 工具的一些行为进行配置。该功能还未完全开发完成，不过其中的一些选项已经可以使用：

- `"version": 0`：配置文件版本，用于对比是否发生不兼容的配置格式版本。
- `"go_install": false`：如果您的包均使用完整的导入路径（例如：`github.com/user/repo/subpkg`）,则可以启用该选项来进行 `go install` 操作，加快构建操作。
- `"watch_ext": []`：用于监控其它类型的文件（默认只监控后缀为 `.go` 的文件）。
- `"dir_structure":{}`：如果您的目录名与默认的 MVC 架构的不同，则可以使用该选项进行修改。
- `"cmd_args": []`：如果您需要在每次启动时加入启动参数，则可以使用该选项。
- `"envs": []`：如果您需要在每次启动时设置临时环境变量参数，则可以使用该选项。
