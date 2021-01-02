---
name: XSRF filtering
sort: 4
---

# Cross-Site Request Forgery

XSRF, [Cross-Site Request Forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery), is an important security concern for web development. Beego has built in XSRF protection which assigns each user a randomized cookie that is used to verify requests.  XSRF protection can be activated by setting `EnableXSRF = true` in the configuration file:

    EnableXSRF = true
    XSRFKey = 61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o
    XSRFExpire = 3600 // set cookie expire in 3600 seconds, default to 60 seconds if not specified

XSRF protection can also be enabled in the main application entry function:

    web.BConfig.WebConfig.EnableXSRF = true
    web.BConfig.WebConfig.XSRFKey = "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o"
    web.BConfig.WebConfig.XSRFExpire = 3600

When XSRF is enabled Beego will set a cookie `_xsrf` for every user. Beego will refuse any `POST`, `PUT`, or `DELETE` request that does not include this cookie. If XSRF protection is enabled a field must be added to provide an `_xsrf` value to every form. This can be added directly in the template with `XSRFFormHTML()`.

A global expiration time should be set using `web.XSRFExpire`. This value can be also be set for individual logic functions:

```go
func (this *HomeController) Get(){
	this.XSRFExpire = 7200
	this.Data["xsrfdata"]=template.HTML(this.XSRFFormHTML())
}
```

**XSRF** works with HTTPS protocol. In Beego 2.x, the cookie storing XSRF token has two flag: [secure](https://en.wikipedia.org/wiki/Secure_cookie) 
and [http-only](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies).

In Beego 1.x (<=1.12.2), we don't have this two flags, so it's not safe because attackers is able to steal the XSRF token.