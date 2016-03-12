---
name: Модуль логирования
sort: 3
---

# Логирование

Модуль логирования вдохновлен `database/sql`. Поддерживает file, console, net и smtp как поставщиков по-умолчанию. Может быть установлен следующим образом:

	go get github.com/astaxie/beego/logs

## Начальное использование

Импортируйте пакет:

	import (
		"github.com/astaxie/beego/logs"
	)

Инициализируйте переменную log (размер кеша 10000):

	log := logs.NewLogger(10000)

Далее добавьте поставщика вывода (поддерживает несколько поставщиков вывода одновременно). Первый параметр - имя поставщика (`console`, `file`, `conn` or `smtp`). Второй параметр - специфичная для поставщика строка конфигурации (об этом смотрите ниже).

	log.SetLogger("console", "")

Затем, мы можем использовать это в нашем коде:

	log.Trace("trace %s %s","param1","param2")
	log.Debug("debug")
	log.Info("info")
	log.Warn("warning")
	log.Error("error")
	log.Critical("critical")

## Логирование информации о месте вызыва (имя файла & номер строки)

Модуль может быть сконфигурирован так, чтобы он добавлял имя файла и номер строки вызова логера в выходном логе. Эта возможность отключена по-умолчанию, но может быть включена следующим образом:

	log.EnableFuncCallDepth(true)

Используйте `true`, чтобы включить 'имя файла & номер строки', и `false`, чтобы выключить. По-умолчанию `false`.

Если ваше приложение инкапсулирует вызов методов логирования, вам, возможно, следует использовать `SetLogFuncCallDepth` to set the number of stack frames to be skipped before the caller information is retrieved. The default is 2.

	log.SetLogFuncCallDepth(3)

## Настройка поставщиков

Каждый провайдер поддерживает набор конфигурационных опций.

- console

	Может настроить уровень логирования или использовать значение по-умолчанию. Используется `os.Stdout` по-умолчанию.

		log := logs.NewLogger(10000)
		log.SetLogger("console", `{"level":1}`)

- file

	E.g.:

		log := logs.NewLogger(10000)
		log.SetLogger("file", `{"filename":"test.log"}`)

	Параметры:
	- filename: Сохранить в файл.
	- maxlines: Максимальное количество строк в лог файле, по-умолчанию - 1000000.
	- maxsize: Максимальный размер каждого лог файла, по-умолчанию - 1 << 28 или 256M.
	- daily: Заменять лог каждый день, по-умолчанию - true.
	- maxdays: Максимальный размер дней которые будут сохраняться в логе, , по-умолчанию - 7.
	- rotate: Включить logrotate, по-умолчанию - true.
	- level: Уровень логирования, по умолчанию - Trace.

- conn

	Логирование в сеть:

		log := NewLogger(1000)
		log.SetLogger("conn", `{"net":"tcp","addr":":7020"}`)

	Параметры:
	- reconnectOnMsg: Если true: Переоткрыть и закрыть соединение каждый раз как сообщение будет отправлено. По-умолчанию - `false`.
	- reconnect: Если `true`: автоматичетски подключаться.  По-умолчанию - `false`.
	- net: тип подключения: tcp, unix или udp.
	- addr: сетевой адрес.
	- level: Уровень логирования, по-умолчанию - Trace.

- smtp

	Логирование c отправкой на email:

		log := logs.NewLogger(10000)
		log.SetLogger("smtp", `{"username":"beegotest@gmail.com","password":"xxxxxxxx","host":"smtp.gmail.com:587","sendTos":["xiemengjun@gmail.com"]}`)

	Параметры:
	- username: smtp пользователь.
	- password: smtp пароль.
	- host: SMTP адрес сервера.
	- sendTos: emails адрес куда будут отправлены логи.
	- subject: email тема, по умолчанию - `Diagnostic message from server`.
	- level: Уровень логирования, по-умолчанию - Trace.
