---
name: Bee tool usage
sort: 2
---

# Introduction to bee tool

Bee tool is a project for help develop beego rapidly. With bee tool we can create, auto compile and reload, develop, test, and deploy beego applications easily.

## Installing bee tool

You can install bee tool through following command:

	go get github.com/beego/bee
	
`bee` is installed into `GOPATH/bin` by default. You need to add `GOPATH/bin` to your PATH, otherwise the `bee` command won't work.

## Bee tool commands

Type `bee` in command line. We can see following messages:

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

### Command `new`

The `new` command can create a new web project. You can create a new beego project by typing `bee new <project name>` under `$GOPATH/src`. It will generate all the project folders and files:

```
bee new myproject
[INFO] Creating application...
/gopath/src/myproject/
/gopath/src/myproject/conf/
/gopath/src/myproject/controllers/
/gopath/src/myproject/models/
/gopath/src/myproject/routers/
/gopath/src/myproject/tests/
/gopath/src/myproject/static/
/gopath/src/myproject/static/js/
/gopath/src/myproject/static/css/
/gopath/src/myproject/static/img/
/gopath/src/myproject/views/
/gopath/src/myproject/conf/app.conf
/gopath/src/myproject/controllers/default.go
/gopath/src/myproject/views/index.tpl
/gopath/src/myproject/routers/router.go
/gopath/src/myproject/tests/default_test.go
/gopath/src/myproject/main.go
13-11-25 09:50:39 [SUCC] New application successfully created!
```

```
myproject
├── conf
│   └── app.conf
├── controllers
│   └── default.go
├── models
├── routers
│   └── router.go
├── tests
│   └── default_test.go
├── static
│   ├── js
│   ├── css
│   └── img
├── views
│   └── index.tpl
├── main.go

11 directories, 5 files
```

### Command `run`

When we develop Go projects, we often have problems of needing to compile and run them manually. The `bee run` command will supervise the file system of any beego project using [inotify](http://en.wikipedia.org/wiki/Inotify) so that we can see the results immediately after any modifications within for beego project folders.

```
cd myproject
bee run
13-11-25 09:53:04 [INFO] Uses 'myproject' as 'appname'
13-11-25 09:53:04 [INFO] Initializing watcher...
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/controllers)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/routers)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/tests)
13-11-25 09:53:04 [INFO] Start building...
13-11-25 09:53:16 [SUCC] Build was successful
13-11-25 09:53:16 [INFO] Restarting myproject ...
13-11-25 09:53:16 [INFO] ./myproject is running...
13-11-25 09:53:16 [app.go:103] [I] http server Running on :8080
```
Upon execution of this command, open your browser and visit `http://localhost:8080/`, you should see your app running:

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

### Command api

The `new` command is used for crafting new web applications. But there are many users who use beego for developing APIs. We can use the `api` command to create new API applications. 
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
│   └── default.go
├── main.go
├── models
│   └── object.go
└── tests
    └── default_test.go
```

Compare this to the `bee new myproject` command seen earlier.
Note that the new API application doesn't have a static and views folder.

### Command test

This is a wrapper for `go test`. It can run the test cases in test folder
of beego project.

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

### Command pack

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

We can see compressed file in the project folder:

```
rwxr-xr-x  1 astaxie  staff  8995376 11 25 22:46 apiproject
-rw-r--r--  1 astaxie  staff  2240288 11 25 22:58 apiproject.tar.gz
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 conf
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 controllers
-rw-r--r--  1 astaxie  staff      509 11 25 22:31 main.go
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 models
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 tests
```

### Command router

This command doesn't work yet. In the future it will generate the routers by analysing the function in controllers.

### Command bale

This command is currently only available to the developer team. It's mainly used for compressing all the static files in to a single binary file. So we don't need to carry  static files including js, css, images and views when publish the project. Those files will be self-extracting with non-overwrite when the program starts.

## Bee tool configuration

You may notice that there is a file named `bee.json` in bee tool source code folder, this file is the configuration file of beego. The full features haven't been done yet, but there are some options you'd like to use for now:

- `"version": 0`: version of file, for checking incompatible format version.
- `"go_install": false`: if you use a full import path like `github.com/user/repo/subpkg`, then you can enable this option to run `go install` and speed up you build processes.
- `"watch_ext": []`: add other file extensions to watch(only watch `.go` files by default). For example, `.ini`, `.conf`, etc.
- `"dir_structure":{}`: if your folder names are not the same as MVC classic namea, you can use this option to change them.
- `"cmd_args": []`: add command arguments for every start.
- `"envs": []`: setting environment variables for every start.
