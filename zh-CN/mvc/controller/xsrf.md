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

    beego.EnableXSRF = true
    beego.XSRFKEY = "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o"
    beego.XSRFExpire = 3600  //过期时间，默认60秒
    

如果开启了 XSRF，那么 beego 的 Web 应用将对所有用户设置一个 `_xsrf` 的 cookie 值（默认过期 60 秒），如果 `POST PUT DELET` 请求中没有这个 cookie 值，那么这个请求会被直接拒绝。如果你开启了这个机制，那么在所有被提交的表单中，你都需要加上一个域来提供这个值。你可以通过在模板中使用 专门的函数 `XSRFFormHTML()` 来做到这一点：

过期时间上面我们设置了全局的过期时间 `beego.XSRFExpire`，但是有些时候我们也可以在控制器中修改这个过期时间，专门针对某一类处理逻辑：

```go
func (this *HomeController) Get(){ 
	this.XSRFExpire = 7200    
	this.Data["xsrfdata"]=template.HTML(this.XSRFFormHTML())
}
```

### 在表单中使用

在 Controller 中这样设置数据：

```go
func (this *HomeController) Get(){        
    this.Data["xsrfdata"]=template.HTML(this.XSRFFormHTML())
}
```
  
然后在模板中这样设置：

    <form action="/new_message" method="post">
      {{ .xsrfdata }}
      <input type="text" name="message"/>
      <input type="submit" value="Post"/>
    </form>

### 在 JavaScript 中使用

如果你提交的是 AJAX 的 POST 请求，你还是需要在每一个请求中通过脚本添加上 _xsrf 这个值。下面是在 AJAX 的 POST 请求，使用了 jQuery 函数来为所有请求都添加 _xsrf 值：

jQuery cookie插件：https://github.com/carhartl/jquery-cookie
base64 插件：http://phpjs.org/functions/base64_decode/

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

#### 扩展 jQuery

通过扩展 ajax 给每个请求加入 xsrf 的 header

需要你在 html 里保存一个 `_xsrf` 值

```go
func (this *HomeController) Get(){        
    this.Data["xsrf_token"] = this.XsrfToken()
}
```

放在你的 head 中

```html
<head>
    <meta name="_xsrf" content="{{.xsrf_token}}" />
</head>
```

扩展 ajax 方法，将 `_xsrf` 值加入 header，扩展后支持 jquery post/get 等内部使用了 ajax 的方法

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

对于 PUT 和 DELETE 请求（以及不使用将 form 内容作为参数的 POST 请求）来说，你也可以在 HTTP 头中以 X-XSRFToken 这个参数传递 XSRF token。

如果你需要针对每一个请求处理器定制 XSRF 行为，你可以重写 Controller 的 CheckXSRFCookie 方法。例如你需要使用一个不支持 cookie 的 API， 你可以通过将 `CheckXSRFCookie()` 函数设空来禁用 XSRF 保护机制。然而如果 你需要同时支持 cookie 和非 cookie 认证方式，那么只要当前请求是通过 cookie 进行认证的，你就应该对其使用 XSRF 保护机制，这一点至关重要。

## 支持controller 级别的屏蔽

XSRF 之前是全局设置的一个参数,如果设置了那么所有的API请求都会进行验证,但是有些时候API 逻辑是不需要进行验证的,因此现在支持在controller 级别设置屏蔽:

```
type AdminController struct{
	beego.Controller
}

func (a *AdminController) Prepare() {
	a.EnableXSRF = false
}
```
