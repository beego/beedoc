# 跨站请求伪造

[跨站请求伪造(Cross-site request forgery)](http://en.wikipedia.org/wiki/Cross-site_request_forgery)， 简称为 XSRF，是Web应用中常见的一个安全问题。前面的链接也详细讲述了XSRF攻击的实现方式。

当前防范XSRF的一种通用的方法，是对每一个用户都记录一个无法预知的cookie数据，然后要求所有提交的请求(POST/PUT/DELETE)中都必须带有这个cookie数据。如果此数据不匹配 ，那么这个请求就可能是被伪造的。

beego有内建的XSRF的防范机制，要使用此机制，你需要在应用配置文件中加上enablexsrf设定：

    enablexsrf = true
    xsrfkey = 61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o
    xsrfexpire = 3600   

或者直接在main入口处这样设置：

    beego.EnableXSRF = true
    beego.XSRFKEY = "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o"
    beego.XSRFExpire = 3600  //过期时间，默认60秒
    

如果开启了XSRF，那么beego的Web应用将对所有用户设置一个`_xsrf`的 cookie 值(默认过期60秒)，如果`POST PUT DELET`请求中没有这 个 cookie 值，那么这个请求会被直接拒绝。如果你开启了这个机制，那么在所有被提交的表单中，你都需要加上一个域来提供这个值。你可以通过在模板中使用 专门的函数 XsrfFormHtml() 来做到这一点：

>>> 过期时间上面我们设置了全局的过去时间`beego.XSRFExpire`，但是有些时候我们也可以在控制器中修改这个过期时间，专门针对某一类处理逻辑

>>>     func (this *HomeController) Get(){ 
>>>        this.XSRFExpire = 7200    
>>>        this.data["xsrfdata"]=template.HTML(this.XsrfFormHtml())
>>>     }

在Controller中这样设置数据

    func (this *HomeController) Get(){        
        this.data["xsrfdata"]=template.HTML(this.XsrfFormHtml())
    }
  
然后在模板中这样设置：

    <form action="/new_message" method="post">
      {{ .xsrfdata }}
      <input type="text" name="message"/>
      <input type="submit" value="Post"/>
    </form>
  
如果你提交的是AJAX的POST请求，你还是需要在每一个请求中通过脚本添加上 _xsrf 这个值。下面是在AJAX的POST请求，使用了jQuery函数来为所有请求组东添加_xsrf值：

    function getCookie(name) {
        var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
        return r ? r[1] : undefined;
    }
    
    jQuery.postJSON = function(url, args, callback) {
        args._xsrf = getCookie("_xsrf");
        $.ajax({url: url, data: $.param(args), dataType: "text", type: "POST",
            success: function(response) {
            callback(eval("(" + response + ")"));
        }});
    };
  
对于 PUT 和 DELETE 请求（以及不使用将form内容作为参数的POST请求）来说，你也可以在 HTTP 头中以 X-XSRFToken 这个参数传递 XSRF token。

如果你需要针对每一个请求处理器定制 XSRF 行为，你可以重写 Controller的CheckXsrfCookie方法。例如你需要使用一个不支持 cookie 的 API， 你可以通过将CheckXsrfCookie() 函数设空来禁用 XSRF 保护机制。然而如果 你需要同时支持 cookie 和非 cookie 认证方式，那么只要当前请求是通过 cookie 进行认证的，你就应该对其使用 XSRF 保护机制，这一点至关重要。
