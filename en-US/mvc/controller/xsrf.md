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

    beego.BConfig.WebConfig.EnableXSRF = true
    beego.BConfig.WebConfig.XSRFKey = "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o"
    beego.BConfig.WebConfig.XSRFExpire = 3600

When XSRF is enabled Beego will set a cookie `_xsrf` for every user. Beego will refuse any `POST`, `PUT`, or `DELETE` request that does not include this cookie. If XSRF protection is enabled a field must be added to provide an `_xsrf` value to every form. This can be added directly in the template with `XSRFFormHTML()`.

A global expiration time should be set using `beego.XSRFExpire`. This value can be also be set for individual logic functions:

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

If AJAX is being used to POST a request, `_xsrf` should be added using javascript. The example below uses jQuery to add `_xsrf` to every request automatically.

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

Save the `_xsrf` value in html:

```go
func (this *HomeController) Get(){
    this.Data["xsrf_token"] = this.XSRFToken()
}
```

Put it into the head:

```html
<head>
    <meta name="_xsrf" content="{{.xsrf_token}}" />
</head>
```
Extending Ajax by adding `_xsrf` into header also supports jQuery methods which use AJAX internally, such as POST and GET.

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

For PUT, POST, and DELETE requests which don't use form content as parameters, pass XSRF tokens by X-XSRFToken in the HTTP header.

To customize XSRF behavior for different requests, overwrite the CheckXSRFCookie method of the Controller. To support an API which doesn't support XSRF, disable XSRF protection by setting `CheckXSRFCookie()` to empty. XSRF protection should be used to accommodate cookie validated requests with and without cookie support at the same time.

## support controller setting

`XSRF` is a global variable.  If it is set it to `true`, then every request will be validated. If the APIs don't need to be validated, then set XSRF to `false` in controllers:

```
type AdminController struct{
	beego.Controller
}

func (a *AdminController) Prepare() {
	a.EnableXSRF = false
}
```
