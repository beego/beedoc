---
name: XSRF 过滤
sort: 4
---

# 跨站请求伪造

[跨站请求伪造(Cross-site request forgery)](http://en.wikipedia.org/wiki/Cross-site_request_forgery)， 简称为 XSRF，是 Web 应用中常见的一个安全问题。前面的链接也详细讲述了 XSRF 攻击的实现方式。

当前防范 XSRF 的一种通用的方法，是对每一个用户都记录一个无法预知的 cookie 数据，然后要求所有提交的请求（POST/PUT/DELETE）中都必须带有这个 cookie 数据。如果此数据不匹配 ，那么这个请求就可能是被伪造的。

beego 有内建的 XSRF 的防范机制，要使用此机制，你需要在应用配置文件中加上 `enablexsrf` 设定：

    enablexsrf = true
    xsrfkey = 61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o
    xsrfexpire = 3600

或者直接在 main 入口处这样设置：

    web.EnableXSRF = true
    web.XSRFKEY = "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o"
    web.XSRFExpire = 3600  //过期时间，默认1小时


如果开启了 XSRF，那么 beego 的 Web 应用将对所有用户设置一个 `_xsrf` 的 cookie 值（默认过期 1 小时），如果 `POST PUT DELET` 请求中没有这个 cookie 值，那么这个请求会被直接拒绝。如果你开启了这个机制，那么在所有被提交的表单中，你都需要加上一个域来提供这个值。你可以通过在模板中使用 专门的函数 `XSRFFormHTML()` 来做到这一点：

过期时间上面我们设置了全局的过期时间 `web.XSRFExpire`，但是有些时候我们也可以在控制器中修改这个过期时间，专门针对某一类处理逻辑：

```go
func (this *HomeController) Get(){
	this.XSRFExpire = 7200
	this.Data["xsrfdata"]=template.HTML(this.XSRFFormHTML())
}
```

在 Beego 2.x 里面有一个很大的不同，就是 Beego 2.x 的XSRF只支持 HTTPS 协议。

这是因为，在 2.x 的时候，我们给存储 XSRF token的 cookie 加上了 [secure](https://en.wikipedia.org/wiki/Secure_cookie), [http-only](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies).
两个设置，所以只能通过 HTTPS 协议运作。

与此同时，你也无法通过 JS 获取到 XSRF token。

这个改进，一个很重要的原因是，在 1.x 的时候，缺乏这两个选项，会导致攻击者可以从 cookie 中拿到 XSRF token，导致 XSRF 失效。


# 支持controller 级别的屏蔽

XSRF 之前是全局设置的一个参数,如果设置了那么所有的 API 请求都会进行验证,但是有些时候API 逻辑是不需要进行验证的,因此现在支持在controller 级别设置屏蔽:

```go
type AdminController struct{
	web.Controller
}

func (a *AdminController) Prepare() {
	a.EnableXSRF = false
}
```
