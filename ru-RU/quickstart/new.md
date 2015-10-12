---
name: Создание нового проекта
sort: 1
---

# Создание проекта

Большинство проектов на BeeGo создаются с помощью команды `bee`. Перед тем как начать, убедитесь, что вы уже установили программу `bee` и пакет `beego`. Если вы пока этого не сделали, прочитайте [Установку beego](../install) и [Установку программы bee](../install/bee.md) перед тем как продолжить.

Когда вы сделаете всё это, можете начинать работу. Откройте терминал, перейдите в директорию `$GOPATH` и введите  `bee new quickstart`:

	➜  src  bee new quickstart
	[INFO] Creating application...
	/gopath/src/quickstart/
	/gopath/src/quickstart/conf/
	/gopath/src/quickstart/controllers/
	/gopath/src/quickstart/models/
	/gopath/src/quickstart/static/
	/gopath/src/quickstart/static/js/
	/gopath/src/quickstart/static/css/
	/gopath/src/quickstart/static/img/
	/gopath/src/quickstart/views/
	/gopath/src/quickstart/conf/app.conf
	/gopath/src/quickstart/controllers/default.go
	/gopath/src/quickstart/views/index.tpl
	/gopath/src/quickstart/main.go
	13-11-26 10:34:10 [SUCC] New application successfully created!
	
Программа bee создаст новый проект BeeGo. Вот его структура:

	quickstart
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

Это типичное приложение MVC. `main.go` - стартовый файл проекта.

## Запуск приложения

После создания проекта мы можем его запустить. Перейдите в папку созданного проекта и введите `bee run`, чтобы запустить его. При этом проект скомпилируется.

	➜  src  cd quickstart
	➜  quickstart  bee run
	13-11-26 10:43:14 [INFO] Uses 'quickstart' as 'appname'
	13-11-26 10:43:14 [INFO] Initializing watcher...
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart/controllers)
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart/models)
	13-11-26 10:43:14 [TRAC] Directory(/gopath/src/quickstart)
	13-11-26 10:43:14 [INFO] Start building...

Мы получили веб-приложение, запущенное на порте `8080` (порт по умолчанию в BeeGo). Удивительно, не правда ли? Вы могли заметить,что мы обошлись без Nginx и Apache. Да,вы правы! В Go уже реализованны все функции, необходимые для сетевого слоя, и BeeGo инкапсулирует их, поэтому на мне нужны Nginx или Apache. Давайте посмотрим, как выглядит наше приложение в браузере:

![](../images/beerun.png)

Вы изумлены? Разве это не легко - создавать веб-приложения? Давайте окунёмся в наш проект и посмотрим, как всё это работает!
