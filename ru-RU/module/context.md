---
name: Модуль Контекст
sort: 5
---

# Модуль Контекст

Контекст - обертка для работы с http запросам и ответами. Модуль Контекст предоставляется входной объект который который является запросоом и выходной объект который ялвяется ответом.

## Объект контекст

Ниже функции которые предоставлены для входного и выходного объекта контекста.
- Redirect
- Abort
- WriteString
- GetCookie
- SetCookie

Объект контекста является параметром для функции Filter, таким образом вы можете использовать фильтр чтобы управлять процессом или завершить его предварительно.

## Входной объект

Входной объект представляет собой обертку запроса. Реализованные методы:

- Protocol

  Получить протокол запроса. E.g.: `HTTP/1.0`
	
- Uri

  RequestURI запроса. E.g.: `/hi`
	
- Url

  URL запроса. E.g.: `http://beego.me/about?username=astaxie`
	
- Site

  Комбинация протокола и домена. E.g.: `http://beego.me`

- Scheme
  
  Схема. E.g.: `http`, `https`
	
- Domain

  Домен. E.g.: `beego.me`
	
- Host

  Домен запроса. Тоже самое что и Domain.
	
- Method

  HTTP метод запроса. Это стандартный HTTP метод. E.g.: `GET`, `POST,
	
- Is

  Проверка HTTP метода. E.g.: `Is('GET')` вернет true или false
	
- IsAjax

  Проверка того что является ли запрос AJAX запросом. Вернет true или false.
	
- IsSecure

  Проверка на что запрос использует https. Вернет true или false.
	
- IsWebsocket

  Проверка того что является ли запрос Websocket запросом. Вернет true или false.
	
- IsUpload

  Провека того что файл был загружен в запросе. Вернет true или false.
	
- IP

  Вернет IP пользователя из запроса. Если пользователь использует прокси, вы получите реальный IP рекурсивно.
	
- Proxy

  Вернет IP всех запросов proxy.
	
- Refer

  Вернет Refer из запроса.
	
- SubDomains

  Вернет родительский домен из запроса. Например, домен в запросе `blog.beego.me`, тогда функция вернет `beego.me`.
	
- Port

  Вернет порт из запроса. E.g.: 8080
	
- UserAgent

  Вернет `UserAgent` из запроса. E.g.: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36`

- Param
  
  Может быть установлент в конфиге router. Используйте что получить эти параметры. E.g.: `Param(":id")` вернут 12
	
- Query

  Вернет все параметры из GET и POST запросов. Это тоже самое что и `$_REQUEST` в PHP
	
- Header

  Вернет заголовки запроса. E.g.: `Header("Accept-Language")` вернет значение из запрашиваемоего заголовка, E.g.: `zh-CN,zh;q=0.8,en;q=0.6`
	
- Cookie

  Вернут куки из запроса. E.g.: `Cookie("username")` вернут значение username из куки
	
- Session

  Вы можете инициализировать сессию. Это объект Session object из мудуля сессии BeeGo. Вернут связанные данные лежащие на серевер.
	
- Body
  
  Вернут тело запроса. E.g.: В API приложениях запрос отправляет JSON данные и они НЕ могут быть извлечены с Query. Вы должны использовать Body чтобы получить JSON данные.
	
- GetData

  Получить значение `Data` в `Input`
	
- SetData			

  Установить значение `Data` в `Input`. `GetData` и `SetData` используются чтобы передать данные из Filter в Controller.
	
## Output Object

Output object is the encapsulation of request. Here are the implemented methods:

- Header

  Set response header. E.g.: `Header("Server","beego")`
	
- Body

  Set response body. E.g.: `Body([]byte("astaxie"))`

- Cookie

  Set response cookie. E.g.: `Cookie("sessionID","beegoSessionID")`
	
- Json

  Parse Data into JSON and call `Body` to return it.
	
- Jsonp

  Parse Data into JSONP and call `Body` to return it.
	
- Xml

  Parse Data into XML and call `Body` to return it.
	
- Download

  Pass in file path and output file.
	
- ContentType

  Set response ContentType
	
- SetStatus

  Set response status
	
- Session

  Set the value will be stored in server. E.g.: `Session("username","astaxie")`. Then it can be read later.
	
- IsCachable

  Test if it's a cacheable status based on status.
	
- IsEmpty

  Test if output is empty based on status.
	
- IsOk

  Test if response is 200 based on status.

- IsSuccessful

  Test if response is successful based on status.
		
- IsRedirect

  Test if response is redirected based on status.

- IsForbidden

  Test if response is forbidden based on status.
	
- IsNotFound

  Test if response is forbidden based on status.
	
- IsClientError

  Test if response is client error based on status.
	
- IsServerError

  Test if response is server error based on status.
