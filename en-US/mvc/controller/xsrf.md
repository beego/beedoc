---
name: XSRF filtering
sort: 4
---

# Cross-site request forgery

[Cross-site request forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery) is well known as XSRF. It is a very important security problem in web development. The wikipedia page talked about it in detail. 

One of the common solutions to prevent XSRF is to record an unpredictable cookie for each user and each request (POST/PUT/DELETE) must have this cookie. If the cookie doesn't match, the request is probably a forged request.

Beego has buildin XSRF protection. If you want to use it, you can either `enablexsrf` in your configuration file:

    enablexsrf = true
    xsrfkey = 61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o
    xsrfexpire = 3600   

or enable it in main enter function:

    beego.EnableXSRF = true
    beego.XSRFKEY = "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o"
    beego.XSRFExpire = 3600  //过期时间，默认60秒
    
With XSRF enabled Beego will set a cookie `_xsrf` (expire in 60 seconds by default) for every user. If this cookie doesn't exist in a `POST`, `PUT`, or `DELETE` request, Beego will refuse the request. If you enabled XSRF protection, you need to add a field to provide `_xsrf` value to every form. You can directly use `XsrfFormHtml()` in the template to set it.

We also set the global expiration time by `beego.XSRFExpire`. We can still change it in controllers for some particular logic:

```go
func (this *HomeController) Get(){ 
	this.XSRFExpire = 7200    
	this.Data["xsrfdata"]=template.HTML(this.XsrfFormHtml())
}
```

### Usage in form

Set data in controller:

```go
func (this *HomeController) Get(){        
    this.Data["xsrfdata"]=template.HTML(this.XsrfFormHtml())
}
```

Then use it in template:  

    <form action="/new_message" method="post">
      {{ .xsrfdata }}
      <input type="text" name="message"/>
      <input type="submit" value="Post"/>
    </form>

### Usage in javascript

If you are using AJAX to POST your request, you still need to add `_xsrf` by javascript. The example below uses jQuery to add `_xsrf` to every request automatically.

jQuery cookie plugin: https://github.com/carhartl/jquery-cookie
base64 plugin: http://phpjs.org/functions/base64_decode/

```js
jQuery.postJSON = function(url, args, callback) {
   var xsrf, xsrflist;
   xsrf = $.cookie("_xsrf");
   xsrflist = xsrf.split("|");
   args._xsrf = base64_decode(xsrflist[0]);
    $.ajax({url: url, data: $.param(args), dataType: "text", type: "POST",
        success: function(response) {
        callback(eval("(" + response + ")"));
    }});
};
```

#### Extending jQuery

Add `xsrf` to every request header by extending ajax.

You need to save the `_xsrf` value in html:

```go
func (this *HomeController) Get(){        
    this.Data["xsrf_token"] = this.XsrfToken()
}
```

You can put it into head:

```html
<head>
    <meta name="_xsrf" content="{{.xsrf_token}}" />
</head>
```
Extending ajax method, adding `_xsrf` into header. This also support jQuery methods which are using ajax internally such as post and get.

```js
var ajax = $.ajax;
$.extend({
    ajax: function(url, options) {
        if (typeof url === 'object') {
            options = url;
            url = undefined;
        }
        options = options || {};
        url = options.url;
        var xsrftoken = $('meta[name=_xsrf]').attr('content');
        var headers = options.headers || {};
        var domain = document.domain.replace(/\./ig, '\\.');
        if (!/^(http:|https:).*/.test(url) || eval('/^(http:|https:)\\/\\/(.+\\.)*' + domain + '.*/').test(url)) {
            headers = $.extend(headers, {'X-Xsrftoken':xsrftoken});
        }
        options.headers = headers;
        return ajax(url, options);
    }
});
```

For PUT and DELETE requests and POST methods which don't use form content as parameters, you can pass XSRF token by X-XSRFToken in HTTP header.

If you need to customize XSRF behavior for different request, you can overwrite CheckXsrfCookie method of Controller. For example you need a API which doesn't support XSRF, you can disable XSRF proction by set `CheckXsrfCookie()` to empty. However, if you need to let the request with cookie support and without cookie support at the same time and current request is validated by cookie, it's important to use XSRF protection.

## support controller setting

`XSRF` is a global varliable, if you set it to true, all the request will validated. but sometimes API don't need validated. So you can controller it in the controllers:

```
type AdminController struct{
	beego.Controller
}

func (a *AdminController) Prepare() {
	a.EnableXSRF = false
}
```

