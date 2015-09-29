# Bee Watch

[![Build Status](https://drone.io/github.com/beego/beewatch/status.png)](https://drone.io/github.com/beego/beewatch/latest)

Bee Watch - интерактивный деббагер для языка программирования Go.

## Возможности

- Три уровня деббагинга: `Critical`, `Info` и `Trace`.
- `Display()` variable values or `Printf()` with customized format.
- Дружественный Web UI для **WebSocket** режима или easy-control **command-line** режим.
- Вызовите `AddWatchVars()` чтобы следилть за переменными и показаывать их информацию когда программа вызвает `Break()`.
- Конфигурационный файл с вашими настройками (`beewatch.json`).

## Установка

Bee Watch доступен через "go get" механизм, вы можете выполнить следующую команду для авто установки:

	go get github.com/beego/beewatch

**Внимание** На данный момент, этот проект может быть установлен только из исходных кодов.

## Быстрый старт

### Использование

	package main

	import (
		"time"

		"github.com/beego/beewatch"
	)

	func main() {
		// Start with default level: Trace,
		// or use `beewatch.Start(beewatch.Info())` to set the level.
		beewatch.Start()

		// Some variables.
		appName := "Bee Watch"
		boolean := true
		number := 3862
		floatNum := 3.1415926
		test := "fail to watch"

		// Add variables to be watched, must be even sized.
		// Note that you have to pass the pointer.
		// For now it only supports basic types.
		// In this case, 'test' will not be watched.
		beewatch.AddWatchVars("test", test, "App Name", &appName,
			"bool", &boolean, "number", &number, "float", &floatNum)

		// Display something.
		beewatch.Info().Display("App Name", appName).Break().
			Printf("boolean value is %v; number is %d", boolean, number)

		beewatch.Critical().Break().Display("float", floatNum)

		// Change some values, must be even sized.
		appName = "Bee Watch2"
		number = 250
		// Here you will see something interesting.
		beewatch.Trace().Break()

		// Multiple goroutines test.
		for i := 0; i < 3; i++ {
			go multipletest(i)
		}

		beewatch.Trace().Printf("Wait 3 seconds...")
		select {
		case <-time.After(3 * time.Second):
			beewatch.Trace().Printf("Done debug")
		}
	
		// Close debugger,
		// it's not useful when you debug a web server that may not have an end.
		beewatch.Close()
	}

	func multipletest(num int) {
		beewatch.Critical().Break().Display("num", num)
	}

### Соединение

Bee Watch деббагер автоматически запускается на[http://localhost:23456](http://localhost:23456) когда вы используете **WebSocket** режим, вы можете изменить порт и другую настройки редактируя `beewatch.json`(скорипируйте дефолтный конфиг из исходной папки Bee Watch).

Ваш браузер должен иметь поддержку WebSocket. Bee Watch был протестирован в Chrome, Safari и Firefox на Mac и Windows.

![Bee Watch demo](https://github.com/beego/beewatch/blob/master/tests/images/demo_beewatch.png?raw=true)

## Примеры и документация API

[Go Walker](http://gowalker.org/github.com/beego/beewatch)

