---
name: Модуль сессий
sort: 1
---

# Введение в модуль сессий

Модуль сессий используется для хранения пользовательских данных между различными запросами. Поддерживается сохранение сессионого идентификатора в куки, таким образом если клиент не поддерживает куки, модуль не будет работать.

Вдохновлен `database/sql`, что значит: один интерфейс, множество реализаций. По умолчанию поддерживает 4 провайдера: memory, file, redis и mysql.

Установка сессионого модуля:

	go get github.com/astaxie/beego/session

## Начальное использование:

Для начала, нужно импортировать пакет:

	import (
		"github.com/astaxie/beego/session"
	)

Затем инициализировать глобальную переменную в качестве менеджера сессий:

	var globalSessions *session.Manager

Затем инициализировать данные в вашей главной функции:

	func init() {
		globalSessions, _ = session.NewManager("memory", `{"cookieName":"gosessionid", "enableSetCookie,omitempty": true, "gclifetime":3600, "maxLifetime": 3600, "secure": false, "sessionIDHashFunc": "sha1", "sessionIDHashKey": "", "cookieLifeTime": 3600, "providerConfig": ""}`)
		go globalSessions.GC()
	}

Параметры NewManager:

1. Имя провайдера: memory, file, mysql, redis
2. JSON строка которая содержит конфигурационную информацию.
	1. cookieName: Имя куки которая хранит сессионный идентификтор у клиента
	2. enableSetCookie, omitempty: Включена ли SetCookie, omitempty
	3. gclifetime: Интервал GC (сборщика мусора).
	4. maxLifetime: Исчетение времени данных сохраненных на севрере
	5. secure: Включена ли https или нет. There is `cookie.Secure` while configure cookie.
	6. sessionIDHashFunc: Функция генератор SessionID. По-умолчанию `sha1`.
	7. sessionIDHashKey: Хеш ключ.
	8. cookieLifeTime: Время истечение куки на клиенте. По-умолчанию 0, что значит что значит время жизни браузера.
	9. providerConfig: Провайдер специцичный конфиг. Смотрите ниже для большей информации.

Затем вы сможете использовать сессию в вашем коде:

	func login(w http.ResponseWriter, r *http.Request) {
		sess, _ := globalSessions.SessionStart(w, r)
		defer sess.SessionRelease(w)
		username := sess.Get("username")
		if r.Method == "GET" {
			t, _ := template.ParseFiles("login.gtpl")
			t.Execute(w, nil)
		} else {
			sess.Set("username", r.Form["username"])
		}
	}

Методы globalSessions:

- `SessionStart` Вернуть объект сессии для текущего запроса.
- `SessionDestroy` Уничтожить текущий сессионный объект.
- `SessionRegenerateId` Сгенирировать новый sessionID.
- `GetActiveSession` Получить активного юзера из сессии.
- `SetHashFunc` Устаовить функцию генератор sessionID.
- `SetSecure` Включить безопасную куку или нет.

Возвращенный объект сессии является интерфейсом. Методы этого интерфейса:

- `Set(key, value interface{}) error`
- `Get(key interface{}) interface{}`
- `Delete(key interface{}) error`
- `SessionID() string`
- `SessionRelease()`
- `Flush() error`

## Конфигурация провайдера сессии

Мы уже видели конфигурацию `memory` провайдера. Ниже конфигурация других провайдеров:

- `mysql`:

	Все переметры такие же как для `memory` кроме четвертого параметра, например:

		username:password@protocol(address)/dbname?param=value

	Подробности можете посмотреть по адресу [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql#dsn-data-source-name).

- `redis`:

	Конфигурация соединения: address,pool,password

		127.0.0.1:6379,100,astaxie

- `file`:

	The session save path. Create new files in two levels by default.  E.g.: if sessionID is `xsnkjklkjjkh27hjh78908` the file will be saved as `./tmp/x/s/xsnkjklkjjkh27hjh78908`

		./tmp

## Создание нового провайдера сессии

Иногда, нам нужно создать собственный провайдер сессии. Сессиойнный модуль использует интерфейс, таким образом вы можете реализовать этото интерфейс чтобы создать ваш собсвенный провайдер довольно просто.


	// SessionStore содержит все данные для одного сессиного процесса с уникальным id.
	type SessionStore interface {
		Set(key, value interface{}) error     // Установить значение в сессию
		Get(key interface{}) interface{}      // Получить значение из сессии
		Delete(key interface{}) error         // Удалить значение из сессии
		SessionID() string                    // Получить текущий session ID
		SessionRelease(w http.ResponseWriter) // Освободиьт ресурсы & сохранить данные в провайдер & вернуть данные
		Flush() error                         // Удалить все данные
	}

	type Provider interface {
		SessionInit(maxlifetime int64, savePath string) error
		SessionRead(sid string) (SessionStore, error)
		SessionExist(sid string) bool
		SessionRegenerate(oldsid, sid string) (SessionStore, error)
		SessionDestroy(sid string) error
		SessionAll() int // Получить все активные сессии
		SessionGC()
	}

В конце, зарегистрируйте ваш провайдер:

	func init() {
		// ownadapter is an instance of session.Provider
		session.Register("own", ownadapter)
	}
