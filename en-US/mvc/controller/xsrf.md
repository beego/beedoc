---
name: XSRF filtering
sort: 4
---

# Cross-site request forgery

[Cross-site request forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery) is well known as XSRF. It is a very important security problem in web development. The Wikipedia page explains it in detail.

One of the most common solutions to prevent XSRF is to record an unpredictable cookie for each user and each request (POST/PUT/DELETE) must have this cookie. If the cookie doesn't match, the request is probably a forged request.

Beego has built-in XSRF protection. If you want to use it, you can either set `enablexsrf = true` in your configuration file:

    enablexsrf = true
    xsrfkey = 61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o
    xsrfexpire = 3600 // set cookie expire in 3600 seconds, default to 60 seconds if not specified

or enable it in the main application entry function:

    beego.EnableXSRF = true
    beego.XSRFKEY = "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o"
    beego.XSRFExpire = 3600

With XSRF enabled, Beego will set a cookie `_xsrf` for every user. If this cookie doesn't exist in a `POST`, `PUT`, or `DELETE` request, Beego will refuse the request. If you enabled XSRF protection, you need to add a field to provide `_xsrf` value to every form. You can directly add `XSRFFormHTML()` in the template to set it.

We also set the global expiration time by `beego.XSRFExpire`. We can still change it in controllers for some particular logic:

```go
func (this *HomeController) Get(){
	this.XSRFExpire = 7200
	this.Data["xsrfdata"]=template.HTML(this.XSRFFormHTML())
}
```

### Usage in form

Set data in controller:

```go
func (this *HomeController) Get(){
    this.Data["xsrfdata"]=template.HTML(this.XSRFFormHTML())
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

Add `xsrf` to every request header by extending AJAX.

You need to save the `_xsrf` value in html:

```go
func (this *HomeController) Get(){
    this.Data["xsrf_token"] = this.XSRFToken()
}
```

You can put it into the head:

```html
<head>
    <meta name="_xsrf" content="{{.xsrf_token}}" />
</head>
```
Extending ajax method, adding `_xsrf` into header. This also supports jQuery methods which are using AJAX internally such as POST and GET.

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

For PUT and DELETE requests and POST requests which don't use form content as parameters, you can pass XSRF token by X-XSRFToken in HTTP header.

If you need to customize XSRF behavior for different requests, you can overwrite the CheckXSRFCookie method of the Controller. For example if you need an API which doesn't support XSRF, you can disable XSRF proction by setting `CheckXSRFCookie()` to empty. However, if you need to accommodate requests with and without cookie support at the same time and the request is validated by cookie, then it's important to use XSRF protection.

## support controller setting

`XSRF` is a global variable,  so if you set it to `true`, then every request will be validated. But sometimes, APIs don't need to be validated, then you can set XSRF to `false` in controllers:

```
type AdminController struct{
	beego.Controller
}

func (a *AdminController) Prepare() {
	a.EnableXSRF = false
}
```
