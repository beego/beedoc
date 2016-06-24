---
name: bee tool usage
sort: 2
---

# Introduction to bee tool

bee tool is a project that helps developing Beego rapidly. With bee tool we can create, auto compile and reload, develop, test, and deploy Beego applications easily.

## Installing bee tool

You can install bee tool by following command:

	go get github.com/beego/bee

`bee` is installed into `GOPATH/bin` by default. You need to add `GOPATH/bin` to your PATH, otherwise the `bee` command won't work.

## bee tool commands

Type `bee` in command line. We can see following messages:

```
bee is a tool for managing Beego framework.

Usage:

	bee command [arguments]

The commands are:

	new         Create a Beego application
	run         run the app and start a Web server for development
	pack        Compress a Beego project into a single file
	api         create an API Beego application
	bale        packs non-Go files to Go source files
	version     show the bee, Beego and Go version
	generate    source code generator
	migrate     run database migrations
```

### Command `new`

The `new` command can create a new web project. You can create a new Beego project by typing `bee new <project name>` under `$GOPATH/src`. It will generate all the project folders and files:

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

### Command `api`

The `new` command is used for crafting new web applications. But there are many users who use Beego for developing APIs. We can use the `api` command to create new API applications.
Here is the result of running `bee api project_name`:

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

Below shows the generated project structure of the new API application:

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

Compare this to the `bee new myproject` command seen earlier.
Note that the new API application doesn't have a `static` and `views` folder.

You can also create model and controller based on the database schema by providing database conn:

`bee api [appname] [-tables=""] [-driver=mysql] [-conn=root:@tcp(127.0.0.1:3306)/test]`

### Command `run`

When we develop Go projects, we often run into the hassle of compiling and running them manually. The `bee run` command will supervise the file system of any Beego project using [inotify](http://en.wikipedia.org/wiki/Inotify) so that we can see the results immediately after any modifications within for Beego project folders and auto-compiling.

```
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
Open your browser and visit `http://localhost:8080/`, you should see your app running:

![](../images/beerun.png)

After modifying the `default.go` file in the `controllers` folder, we can see the output below from the command line:

```
13-11-25 10:11:20 [EVEN] "/gopath/src/myproject/controllers/default.go": DELETE|MODIFY
13-11-25 10:11:20 [INFO] Start building...
13-11-25 10:11:20 [SKIP] "/gopath/src/myproject/controllers/default.go": CREATE
13-11-25 10:11:23 [SKIP] "/gopath/src/myproject/controllers/default.go": MODIFY
13-11-25 10:11:23 [SUCC] Build was successful
13-11-25 10:11:23 [INFO] Restarting myproject ...
13-11-25 10:11:23 [INFO] ./myproject is running...
```

Refreshing the browser should show the results of the new modifications.


### Command `pack`

`pack` command is used to compress the project into a single file. Then we can deploy the project by uploading and extracting the zip file to the server.

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

We can see the compressed file in the project folder:

```
rwxr-xr-x  1 astaxie  staff  8995376 11 25 22:46 apiproject
-rw-r--r--  1 astaxie  staff  2240288 11 25 22:58 apiproject.tar.gz
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 conf
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 controllers
-rw-r--r--  1 astaxie  staff      509 11 25 22:31 main.go
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 models
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 tests
```

### Command `bale`

This command is currently only available to the developer team. It's mainly used for compressing all the static files in to a single binary file. So we don't need to carry static files including js, css, images and views when publish the project. Those files will be self-extracting with non-overwrite when the program starts.

### Command `version`

This command displays the version of `bee`, `beego`, and `go`.

```
$ bee version
bee   :1.2.2
Beego :1.4.2
Go    :go version go1.3.3 darwin/amd64
```

### Command `generate`
This command will generate the routers by analyzing the functions in controllers.


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


### Command `migrate`
This command will run database migration scripts.

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


## bee tool configuration

You may notice that there is a file named `bee.json` in bee tool source code folder, this file is the configuration file of Beego. The full features haven't been done yet, but there are some options you'd like to use for now:

- `"version": 0`: version of file, for checking incompatible format version.
- `"go_install": false`: if you use a full import path like `github.com/user/repo/subpkg`, then you can enable this option to run `go install` and speed up you build processes.
- `"watch_ext": []`: add other file extensions to watch(only watch `.go` files by default). For example, `.ini`, `.conf`, etc.
- `"dir_structure":{}`: if your folder names are not the same as MVC classic names, you can use this option to change them.
- `"cmd_args": []`: add command arguments for every start.
- `"envs": []`: setting environment variables for every start.
