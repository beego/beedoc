---
name: context 模块
sort: 5
---

# 上下文模块

上下文模块主要是针对 HTTP 请求中，request 和 response 的进一步封装，他包括用户的输入和输出，用户的输入即为 request，context 模块中提供了 Input 对象进行解析，用户的输出即为 response，context 模块中提供了 Output 对象进行输出。

## context 对象

context 对象是对 Input 和 Output 的封装，里面封装了几个方法：
- Redirect
- Abort
- WriteString
- GetCookie
- SetCookie

context 对象是 Filter 函数的参数对象，这样你就可以通过 filter 来修改相应的数据，或者提前结束整个的执行过程。

## Input 对象

Input 对象是针对 request 的封装，里面通过 reqeust 实现很多方便的方法，具体如下：

- Protocol

	获取用户请求的协议，例如 `HTTP/1.0`
	
- Uri

	用户请求的 RequestURI，例如 `/hi?id=1001`
	
- Url

	请求的 URL 地址，例如 `/hi`
	
- Site

	请求的站点地址,scheme+doamin 的组合，例如 `http://beego.me`
	
- Scheme

	请求的 scheme，例如 "http" 或者 "https"
	
- Domain

	请求的域名，例如 `beego.me`
	
- Host

	请求的域名，和 domain 一样
	
- Method

	请求的方法，标准的 HTTP 请求方法，例如 `GET`、`POST` 等	
- Is

	判断是否是某一个方法，例如 `Is("GET")` 返回 true
	
- IsAjax

	判断是否是 AJAX 请求，如果是返回 true，不是返回 false
	
- IsSecure

	判断当前请求是否 HTTPS 请求，是返回 true，否返回 false
	
- IsWebsocket

	判断当前请求是否 Websocket 请求，如果是返回 true，否返回 false
	
- IsUpload

	判断当前请求是否有文件上传，有返回 true，否返回 false
	
- IP

	返回请求用户的 IP，如果用户通过代理，一层一层剥离获取真实的 IP
	
- Proxy

	返回用户代理请求的所有 IP
	
- Refer

	返回请求的 refer 信息
	
- SubDomains

	返回请求域名的根域名，例如请求是 `blog.beego.me`，那么调用该函数返回 `beego.me`
	
- Port

	返回请求的端口，例如返回 8080
	
- UserAgent

	返回请求的 `UserAgent`，例如 `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36`

- Param
	
	在路由设置的时候可以设置参数，这个是用来获取那些参数的，例如 `Param(":id")`,返回12
	
- Query

	该函数返回 Get 请求和 Post 请求中的所有数据，和 PHP 中 `$_REQUEST` 类似
	
- Header

	返回相应的 header 信息，例如 `Header("Accept-Language")`，就返回请求头中对应的信息 `zh-CN,zh;q=0.8,en;q=0.6`
	
- Cookie

	返回请求中的 cookie 数据，例如 `Cookie("username")`，就可以获取请求头中携带的 cookie 信息中 username 对应的值
	
- Session

	session 是用户可以初始化的信息，默认采用了 beego 的 session 模块中的 Session 对象，用来获取存储在服务器端中的数据。
	
- Body

	返回请求 Body 中数据，例如 API 应用中，很多用户直接发送 json 数据包，那么通过 Query 这种函数无法获取数据，就必须通过该函数获取数据。
	
- GetData

	用来获取 Input 中 Data 中的数据
	
- SetData			

	用来设置 Input 中 Data 的值，上面 GetData 和这个函数都是用来方便用户在 Filter 中传递数据到 Controller 中来执行
	
## Output 对象

Output 是针对 Response 的封装，里面提供了很多方便的方法：

- Header

	设置输出的 header 信息，例如 `Header("Server","beego")`
	
- Body

	设置输出的内容信息，例如 `Body([]byte("astaxie"))`

- Cookie

	设置输出的 cookie 信息，例如 `Cookie("sessionID","beegoSessionID")`
	
- Json

	把 Data 格式化为 Json，然后调用 Body 输出数据
	
- Jsonp

	把 Data 格式化为 Jsonp，然后调用 Body 输出数据
	
- Xml

	把 Data 格式化为 Xml，然后调用 Body 输出数据
	
- Download

	把 file 路径传递进来，然后输出文件给用户
	
- ContentType

	设置输出的 ContentType
	
- SetStatus

	设置输出的 status
	
- Session

	设置在服务器端保存的值，例如 `Session("username","astaxie")`，这样用户就可以在下次使用的时候读取
	
- IsCachable

	根据 status 判断，是否为缓存类的状态
	
- IsEmpty

	根据 status 判断，是否为输出内容为空的状态
	
- IsOk

	根据 status 判断，是否为 200 的状态

- IsSuccessful

	根据 status 判断，是否为正常的状态
		
- IsRedirect

	根据 status 判断，是否为跳转类的状态

- IsForbidden

	根据 status 判断，是否为禁用类的状态
	
- IsNotFound

	根据 status 判断，是否为找不到资源类的状态
	
- IsClientError

	根据 status 判断，是否为请求客户端错误的状态
	
- IsServerError

	根据 status 判断，是否为服务器端错误的状态
