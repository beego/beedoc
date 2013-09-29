# Cross-site request forgery

[Cross-site request forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery), or XSRF, is a common problem for web applications. See the Wikipedia article for more information on how XSRF works.

The generally accepted solution to prevent XSRF is to cookie every user with an unpredictable value and include that value as an additional argument with every form submission(POST/GET/DELETE) on your site. If the cookie and the value in the form submission do not match, then the request is likely forged.

beego comes with built-in XSRF protection. To include it in your site, set the following in your application's app.conf:

    enablexsrf = true
    xsrfkey = 61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o
    xsrfexpire = 3600

or set in your application's main.go:

    beego.EnableXSRF = true
    beego.XSRFKEY = "61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o"
    beego.XSRFExpire = 3600  //cookie expired time，default 60s

If enablexsrf is set, the beego web application will set the _xsrf cookie for all users and reject all POST, PUT, and DELETE requests that do not contain a valid _xsrf value. If you turn this setting on, you need to instrument all forms that submit via POST to contain this field. You can do this with the special function `XsrfFormHtml()`, available in all templates. Sometimes you can modify the cookie expired time in the controller, but not affect other controller's logic.

```go
func (this *HomeController) Get(){
	this.XSRFExpire = 7200
	this.data["xsrfdata"] = template.HTML(this.XsrfFormHtml())
}
```

first in controller you get the csrfdata:

```go
func (this *HomeController) Get(){
    this.data["xsrfdata"] = template.HTML(this.XsrfFormHtml())
}
```

then write your template like this:

    <form action="/new_message" method="post">
      {{ .xsrfdata }}
      <input type="text" name="message"/>
      <input type="submit" value="Post"/>
    </form>

If you submit AJAX POST requests, you will need to instrument your JavaScript to include the _xsrf value with each request. This is the jQuery function AJAX POST requests that automatically adds the _xsrf value to all requests:

```javascript
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
```

For PUT and DELETE requests (as well as POST requests that do not use form-encoded arguments), the XSRF token may also be passed via an HTTP header named X-XSRFToken.

If you need to customize XSRF behavior on a per-handler basis, you can override `Controller.CheckXsrfCookie()`. For example, if you have an API whose authentication does not use cookies, you may want to disable XSRF protection by making `CheckXsrfCookie()` do nothing. However, if you support both cookie and non-cookie-based authentication, it is important that XSRF protection be used whenever the current request is authenticated with a cookie.
