---
name: Модуль конфигурации
sort: 7
---

# Разбираем конфигурационный файл

Модуль конфигурации используется для парсинга конфигурационных файлов, вдохновленный `database/sql`. Он поддерживает: ini, json, xml and yaml файлы. Установить его можно так:

	go get github.com/astaxie/beego/config

Если вы хотите распарсить xml или yaml, вы должны сначала установить:

	go get -u github.com/astaxie/beego/config/xml

и потом импортировать:

	import _ "github.com/astaxie/beego/config/xml"

## Простое использование

Инициализируем объект для парсинга:

	iniconf, err := NewConfig("ini", "testini.conf")
	if err != nil {
		t.Fatal(err)
	}

Получаем данные:

	iniconf.String("appname")

### Методы парсера

Тут методы парсера:

* **Установка значений:**
	* `Set(key, val string) error`
	* `SaveConfigFile(filename string) error`
* **Получение значений:**
	* `String(key string) string`
	* `Strings(key string) []string`
	* `Int(key string) (int, error)`
	* `Int64(key string) (int64, error)`
	* `Bool(key string) (bool, error)`
	* `Float(key string) (float64, error)`
	* `DIY(key string) (interface{}, error)`
	* `GetSection(section string) (map[string]string, error)`
* **Получение значений или значения по-умолчанию:**
	* `DefaultString(key string, defaultval string) string`
	* `DefaultStrings(key string, defaultval []string) []string`
	* `DefaultInt(key string, defaultval int) int`
	* `DefaultInt64(key string, defaultval int64) int64`
	* `DefaultBool(key string, defaultval bool) bool`
	* `DefaultFloat(key string, defaultval float64) float64`

### Секции конфигурации

Файлый конфигурации поддерживают секции. Вы можете получить значения внутри секции так `section::key`.

Для примера:

	[demo]
	key1 = "asta"
	key2 = "xie"

Вы можете использовать `iniconf.String("demo::key2")` для получение значения.
