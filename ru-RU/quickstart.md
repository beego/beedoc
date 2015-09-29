# Быстрый старт

## Установка

BeeGo содержит тестовое приложение на котором вы можете изучить как использовать BeeGo app framework.

Вам нужен установленный Go 1.1 для того что бы запустить BeeGo.

Вам нужно установить BeeGo и иснтрумент разработчика [Bee](http://beego.me/docs/install/bee.md):

	$ go get github.com/astaxie/beego
	$ go get github.com/beego/bee


Для удобства, вы должны добавить `$GOPATH/bin` в вашу переменную окружения `$PATH`.

Хотите быстро увидеть как это работает? Тогда сделайте пару простых шагов:

	$ cd $GOPATH/src
	$ bee new hello
	$ cd hello
	$ bee run

Windows пользователи

    > cd %GOPATH%/src
    > bee new hello
    > cd hello
    > bee run hello

Эти команды помогут вам:

1. Установите BeeGo в ваш `$GOPATH`.
2. Установите Bee tool на ваш компьютер.
3. Создайте новое приложение с именем `hello`.
4. Скомпилируйте.

После того как BeeGo запустится, откройте в браузере [http://localhost:8080/](http://localhost:8080/).

## Простой пример

Этот пример выведет `Hello world` в браузер, это показывает насколько просто можно сделать веб-приложение на BeeGo.

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

Сохраните файл как `hello.go`, скомпилируйте и запустите:

	$ go build -o hello hello.go
	$ ./hello

Откройте [http://127.0.0.1:8080](http://127.0.0.1:8080) в браузере и вы должны будете увидеть `hello world`.

Что происходит в приведенном выше примере?

1. Мы импортируем пакет `github.com/astaxie/beego`. Как вы знаете, Go запускает метод init() при инициализации каждого пакета ([больше деталей об этом](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/02.3.md#main-function-and-init-function)), так вот beego инициализирует приложение именно в этот момент.
2. Создаем контроллер. Мы создаем структуру названную `MainController` с анонимным полем `beego.Controller`,
и таким образом `MainController` получает все методы которые имеет `beego.Controller`.
3. Создаем RESTful метод. Благодаря анонимному полю, про которое было сказано выше, `MainController` уже имеет методы `Get`, `Post`, `Delete`, `Put`. Эти методы будут вызваны когда пользователь пошлет соответствующий запрос (к примеру: метод `Post` будет вызван для обработки POST запроса). Поэтому, мы перегрузили метод `Get` в `MainController` и теперь все GET запросы будут использовать этот метод в `MainController` вместо `beego.Controller`.
4. Создаем главный метод. Все приложения в Go используют `main` как точку вхождения подобно тому как это делается в C.
5. Регистрируем роуты. Они сообщают BeeGo какой контроллер будет отвечать на конкретный запрос. Здесь мы регистрируем `MainController` для `/`, и после этого все запросы на `/` будут обработаны `MainController`. Имейте в виду что первый аргумент это путь, а второй аргумент это указатель на контроллер который вы хотите зарегистрировать.
6. Запускаем приложение на порту по-умолчанию 8080. Если хотите закрыть приложение, то нажмите `Ctrl+c`.

Following are shortcut `.bat` files for Windows users:

Создайте два файла `step1.install-bee.bat` и `step2.new-beego-app.bat` в `%GOPATH%/src`.

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

Запустите эти два файла для быстрого старта в вашем изучении BeeGo. А в дальнейшем просто запускайте `run.bat`, если хотите запустить BeeGo снова.
