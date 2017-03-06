---
name: Инструменты Bee tool
sort: 2
---

# Введение в инструменты разработки bee tool

Bee tool - это проект, который помогает разрабатывать быстрее с BeeGo. С bee tool мы можем создавать, автоматически компилировать, перезагружать, тестировать и деплоить приложения проще.

## Устанавливаем bee tool

Вы можете установить bee tool используя следующую команду:

	go get github.com/beego/bee

`bee` будет установлен в `GOPATH/bin` по умолчанию. Вам надо добавить `GOPATH/bin` в вашу переменную окружения PATH, в противном случае команда `bee` не будет работать.

## Bee tool команды

Наберите `bee` в консоли. Вы должны увидеть следующее:

```
Bee is a tool for managing BeeGo framework.

Usage:

	bee command [arguments]

The commands are:

    new         create an application base on beego framework
    run         run the app which can hot compile
    pack        compress an beego project
    api         create an api application base on beego framework
    router      auto-generate routers for the app controllers
    bale        packs non-Go files to Go source files
```

### Команда new

`new` создает новый веб-проект. Вы можете создать новый BeeGo проект набрав `bee new <project name>` находясь в `$GOPATH/src`. Это сгенерирует все проектные папки и файлы:

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

### Команда run

Когда мы разрабатываем проект на Go, мы часто имеем проблему с потребностью постоянно компилировать и перезапускать бинарник приложения вручную. `bee run` будет следить за файловой системой в вашем beego проекте используя inotify. Таким образом мы можем видеть результат сразу же после внесения изменений в проекте.

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
Откройте браузер и перейдите на `http://localhost:8080/`, и вы должны увидеть изменения:

![](../images/beerun.png)

После изменения `default.go` в папке `controllers`, мы можем видеть
подобный вывод в консоли:

```
13-11-25 10:11:20 [EVEN] "/gopath/src/myproject/controllers/default.go": DELETE|MODIFY
13-11-25 10:11:20 [INFO] Start building...
13-11-25 10:11:20 [SKIP] "/gopath/src/myproject/controllers/default.go": CREATE
13-11-25 10:11:23 [SKIP] "/gopath/src/myproject/controllers/default.go": MODIFY
13-11-25 10:11:23 [SUCC] Build was successful
13-11-25 10:11:23 [INFO] Restarting myproject ...
13-11-25 10:11:23 [INFO] ./myproject is running...
```

Теперь обновите браузер и вы сможете увидеть что результат ваших модификаций уже здесь.

### Команда api

Команда `new` используется для создания нового веб-проекта. Но есть много разработчиков, 
которые используют beego для создания API приложений. Вы можете использовать команду `api` для создания API приложения. Вот результат запуска `bee api project_name`:

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

Получившаяся структура API приложения:

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

В сравнении с веб-приложением, приложение для API не содержит файлы статики и шаблоны, но
имеет модуль для запуска unit тестирования.


### Команда pack

Команда `pack` используется для запаковки вашего проекта в один файл. Вы можете использовать это при деплое вашего приложения, для загрузки и извлечения содержимого архива на сервере.

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

Вы можете увидеть архив в корне вашего проекта:

```
rwxr-xr-x  1 astaxie  staff  8995376 11 25 22:46 apiproject
-rw-r--r--  1 astaxie  staff  2240288 11 25 22:58 apiproject.tar.gz
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 conf
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 controllers
-rw-r--r--  1 astaxie  staff      509 11 25 22:31 main.go
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 models
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 tests
```

### Команда router

На данный момент эта команда еще не работает. В будущем она будет генерировать роуты на основе анализа методов в контроллерах.

### Команда bale

Эта команда доступна только девелоперам на данный момент. Её основное назначение - это сжатие всей статики в один бинарный файл. Таким образом вам не нужно волноваться о доставке js, css файлов, изображений и шаблонов когда вы деплоите проект. Эти файлы будут автоматически извлечены с невозможностью перезаписи, когда вы запустите вашу программу.

## Bee tool конфигурация

Вы можете заметить, что в исходных кодах bee tool есть файл с именем `bee.json`. Это файл конфигурации BeeGo. Полный список его возможностей всё еще не существует, но вот некоторые опции которые вы можете использовать уже сейчас:

- `"version": 0`: version of file, for checking incompatible format version.
- `"go_install": false`: if you use full import path like `github.com/user/repo/subpkg`, then you can enable this opetion to run `go install` and speed up you build processes.
- `"watch_ext": []`: add other file extensions to watch(only watch `.go` files by default). For example, `.ini`, `.conf`, etc.
- `"dir_structure":{}`: if your folders' name are not same as MVC classic name, you can use this option use change them.
- `"cmd_args": []`: add command arguments for every start.
- `"envs": []`: setting environment variables for every start.
