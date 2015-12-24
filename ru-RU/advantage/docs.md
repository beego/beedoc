---
name: Автоматическое создание документации для API
sort: 2
---

# Автоматическое создание документации для API

Автоматическая документация это очень крутая возможность которую мы хотели иметь. Сейчас это стало реальностью с BeeGo. Как я сказал, BeeGo не только ускоряет разработку API, но также делает API легко используемым для пользователя. Хорошо, давайте попробуем создать документацию, для этого вам надо просто создать новое API приложение с помощью команды `bee api beeapi`.

# Глобальные настройки API

Добавьте комментарий в самый верх `routers/router.go`:

```
// @APIVersion 1.0.0
// @Title mobile API
// @Description mobile has every tool to get any job done, so codename for the new mobile APIs.
// @Contact astaxie@gmail.com
package routers
```

Этот комментарий устанавливает глобальную информацию. Вот список доступных параметров:

- @APIVersion
- @Title
- @Description
- @Contact
- @TermsOfServiceUrl
- @License
- @LicenseUrl

## Разбор путей
Прямо сейчас, создание автоматической документации для API поддерживает только namespace + Include и парсит только первые два уровня вложенности. Первый уровень это версия API, а второй уровень это модули.

```
func init() {
	ns :=
		beego.NewNamespace("/v1",
			beego.NSNamespace("/customer",
				beego.NSInclude(
					&controllers.CustomerController{},
					&controllers.CustomerCookieCheckerController{},
				),
			),
			beego.NSNamespace("/catalog",
				beego.NSInclude(
					&controllers.CatalogController{},
				),
			),
			beego.NSNamespace("/newsletter",
				beego.NSInclude(
					&controllers.NewsLetterController{},
				),
			),
			beego.NSNamespace("/cms",
				beego.NSInclude(
					&controllers.CMSController{},
				),
			),
			beego.NSNamespace("/suggest",
				beego.NSInclude(
					&controllers.SearchController{},
				),
			),
		)
	beego.AddNamespace(ns)
}
```

## Комментарии приложения
Очень важная вещь это комментарии. Для примера:

```
package controllers

import "github.com/astaxie/beego"

// CMS API
type CMSController struct {
	beego.Controller
}

func (c *CMSController) URLMapping() {
	c.Mapping("StaticBlock", c.StaticBlock)
	c.Mapping("Product", c.Product)
}

// @Title getStaticBlock
// @Description get all the staticblock by key
// @Param	key		path 	string	true		"The email for login"
// @Success 200 {object} models.ZDTCustomer.Customer
// @Failure 400 Invalid email supplied
// @Failure 404 User not found
// @router /staticblock/:key [get]
func (c *CMSController) StaticBlock() {

}

// @Title Get Product list
// @Description Get Product list by some info
// @Success 200 {object} models.ZDTProduct.ProductList
// @Param	category_id		query	int	false		"category id"
// @Param	brand_id	query	int	false		"brand id"
// @Param	query	query	string 	false		"query of search"
// @Param	segment	query	string 	false		"segment"
// @Param	sort 	query	string 	false		"sort option"
// @Param	dir 	query	string 	false		"direction asc or desc"
// @Param	offset 	query	int		false		"offset"
// @Param	limit 	query	int		false		"count limit"
// @Param	price 			query	float		false		"price"
// @Param	special_price 	query	bool		false		"whether this is special price"
// @Param	size 			query	string		false		"size filter"
// @Param	color 			query	string		false		"color filter"
// @Param	format 			query	bool		false		"choose return format"
// @Failure 400 no enough input
// @Failure 500 get products common error
// @router /products [get]
func (c *CMSController) Product() {

}
```

Мы написали комментарий выше для `CMSController` который будет показывать этот модуль. Теперь нам нужно написать
комментарии для каждой функции. Ниже представлены поддерживаемые опции:

- @Title

	Название для этого API. Это строка, весь контент после первого пробела будет считан как название

- @Description

	Описание для этого API. Это строка, весь контент после первого пробела будет считан как описание

- @Param

	`@Param` определяет какие параметры надо послать серверу. Имеется пять колонок для каждого `@Param`:
	1. название параметра;
	2. тип параметра; Это может быть `form`, `query`, `path`, `body` или `header`. `form` означает что параметр посылается как POST. `query` означает что параметр передается в URL как GET. `path` означает что параметр это часть URL. `body` означает что параметр передается как raw data в теле запроса. `header` означает что параметр присутствует в заголовках запроса.
	3. тип данных параметра;
	4. обязательный;
	5. комментарий;

- @Success

	Сообщение которое будет возвращено на клиент в случае успеха. Три параметра.
	1. Код статуса.
	2. Возвращаемый тип; Может быть обёрнут в {}.
	3. Возвращаемые данные; Может быть объектом или строкой. Для {object}, используйте путь и название объекта в вашем проекте и `bee` tool будет смотреть прямиком в объект когда будет создавать документацию. Для примера `models.ZDTProduct.ProductList` представляет объект `ProductList` из `/models/ZDTProduct`

	>>> Используйте пробелы для разделения этих параметров

- @Failure

	Сообщение которое будет возвращено на клиент в случае неудачи. Два параметра.
	1. Код статуса.
	2. Сообщение об ошибке.

	>>> Используйте пробелы для разделения этих параметров

- @router

	Информация о роутинге. Два параметра.
	1. Адрес запроса
	2. Поддерживаемые методы запроса. Оборачивается в `[]`. Используйте `,` для разделения методов если их несколько.

	>>> Используйте пробелы для разделения этих параметров

## Создаем документацию автоматически

Что бы заставить это работать сделайте несколько шагов:

1. Включите документацию задав `EnableDocs = true` в `conf/app.conf`
2. Импортируйте `_ "beeapi/docs"` в `main.go`
3. Используйте `bee run -downdoc=true -gendoc=true` для запуска вашего API приложения и ребилда документации
4. Откройте `swagger document from API project's URL and port. (see item #1 below)`

Документация вашего API готова. Откройте браузер и проверьте результат.

![](../images/docs.png)

![](../images/doc_test.png)

## Проблемы которые могут возникнуть
1. CORS
	Есть два решения
	1. Встройте `swagger` в ваше приложение. Скачайте [swagger](https://github.com/beego/swagger/releases) и положите его в папку проекта. (`bee run -downdoc=true` эта команда может сделать тоже самое, скачать и положить в ваш проект нужные файлы)
	И добавьте эти строки после функции `beego.Run()` в `func main()` в файле `main.go`

		if beego.RunMode == "dev" {
			beego.DirectoryIndex = true
			beego.StaticDir["/swagger"] = "swagger"
		}

	Теперь зайдите на `swagger` документацию.
	2. Сделайте API поддерживающим CORS

			ctx.Output.Header("Access-Control-Allow-Origin", "*")

2. Другие проблемы. Это используется в проектах разработчиков BeeGo, но если у вас будут проблемы неописанные здесь, пожалуйста создайте issues для нас.
