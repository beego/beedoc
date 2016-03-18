---
name: Context Module
sort: 5
---

# Context Module

The Context module is an encapsulation for http request and response. The Context module provides an Input object for user input which is the request and an Output object for output which is the response.

## Context Object

Here are the functions encapsulated for input and output in the context object.
- Redirect
- Abort
- WriteString
- GetCookie
- SetCookie

Context object is the parameter of a Filter function so that you can use a filter to manipulate it or finish the process in advance.

## Input Object

The Input object is the encapsulation of request. Here are the implemented methods:

- Protocol

  Get request protocol. E.g.: `HTTP/1.0`
	
- Uri

  The RequestURI of request. E.g.: `/hi`
	
- Url

  The URL of request. E.g.: `http://beego.me/about?username=astaxie`
	
- Site

  The combination of scheme and domain. E.g.: `http://beego.me`

- Scheme
  
  The request scheme. E.g.: `http`, `https`
	
- Domain

  The request domain. E.g.: `beego.me`
	
- Host

  The request domain. Same as Domain.
	
- Method

  The request method. It's a standard http request method. E.g.: `GET`, `POST`,
	
- Is

  Test if it's a http method. E.g.: `Is('GET')` will return true or false
	
- IsAjax

  Test if it's a ajax request. Return true or false.
	
- IsSecure

  Test if the request is an https request. Return true or false.
	
- IsWebsocket

  Test if the request is a Websocket request. Return true or false.
	
- IsUpload

  Test if there a is file uploaded in the request. Return true or false.
	
- IP

  Return the IP of the requesting user. If the user is using a proxy, it will get the real IP recursively.
	
- Proxy

  Return all IP addresses of the proxy request.
	
- Refer

  Return the refer of the request.
	
- SubDomains

  Return the root domain of request. For example, request domain is `blog.beego.me`, then this function returns `beego.me`.
	
- Port

  Return the port of request. E.g.: 8080
	
- UserAgent

  Return `UserAgent` of request. E.g.: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36`

- Param
  
  Can be set in router config. Used to get those params. E.g.: `Param(":id")` return 12
	
- Query

  Return all params in GET and POST requests. This is similar as `$_REQUEST` in PHP
	
- Header

  Return request header. E.g.: `Header("Accept-Language")` will return the value in request header, E.g.: `zh-CN,zh;q=0.8,en;q=0.6`
	
- Cookie

  Return request Cookie. E.g.: `Cookie("username")` will return the value of username in cookies
	
- Session

  Return initialized session. It is the Session object in the session module of Beego. Return the related data stored on the server.
	
- Body
  
  Return request body. E.g.: in API application request sends JSON data and it can't be retrieved by Query. You need to use Body to get the JSON data.
	
- GetData

  Get value of `Data` in `Input`
	
- SetData			

  Set value of `Data` in `Input`. `GetData` and `SetData` is used to pass data from Filter to Controller.
	
## Output Object

Output object is the encapsulation of response. Here are the implemented methods:

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

  Set the value which will be stored on the server. E.g.: `Session("username","astaxie")`. Then it can be read later.
	
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
