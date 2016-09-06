---
name: 过滤器
sort: 5
---

# 过滤器

beego 支持自定义过滤中间件，例如安全验证，强制跳转等。

过滤器函数如下所示：

	beego.InsertFilter(pattern string, postion int, filter FilterFunc, params ...bool)

InsertFilter 函数的三个必填参数，一个可选参数

- pattern 路由规则，可以根据一定的规则进行路由，如果你全匹配可以用 `*`
- postion 执行 Filter 的地方，四个固定参数如下，分别表示不同的执行过程
	- BeforeRouter 寻找路由之前
	- BeforeExec 找到路由之后，开始执行相应的 Controller 之前
	- AfterExec 执行完 Controller 逻辑之后执行的过滤器
	- FinishRouter 执行完逻辑之后执行的过滤器
- filter filter 函数 type FilterFunc func(*context.Context)
- params 
  1. 设置 returnOnOutput 的值(默认true), 是否允许如果有输出是否跳过其他filters，默认只要有输出就不再执行其他filters
  2. 是否重置filters的参数，默认是false，因为在filters的pattern和本身的路由的pattern冲突的时候，可以把filters的参数重置，这样可以保证在后续的逻辑中获取到正确的参数，例如设置了`/api/*`的filter，同时又设置了`/api/docs/*`的router，那么在访问`/api/docs/swagger/abc.js`的时候，在执行filters的时候设置`:splat`参数为`docs/swagger/abc.js`，但是如果不清楚filter的这个路由参数，就会在执行路由逻辑的时候保持`docs/swagger/abc.js`，如果设置了true，就会重置`:splat`参数.

>>> AddFilter 从beego1.3版本开始已经废除

如下例子所示，验证用户是否已经登录，应用于全部的请求：

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok && ctx.Request.RequestURI != "/login" {
        ctx.Redirect(302, "/login")
    }
}

beego.InsertFilter("/*",beego.BeforeRouter,FilterUser)
```

>>>这里需要特别注意使用 session 的 Filter 必须在 BeforeStatic 之后才能获取，因为 session 没有在这之前初始化。

还可以通过正则路由进行过滤，如果匹配参数就执行：

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}
beego.InsertFilter("/user/:id([0-9]+)",beego.BeforeRouter,FilterUser)
```
## 过滤器实现路由
beego1.1.2开始Context.Input中增加了RunController和RunMethod,这样我们就可以在执行路由查找之前,在filter中实现自己的路由规则.

如下示例实现了如何实现自己的路由规则:

```go
var UrlManager = func(ctx *context.Context) {
    //数据库读取全部的url mapping数据
	urlMapping := model.GetUrlMapping()
	for baseurl,rule:=range urlMapping {
		if baseurl == ctx.Request.RequestURI {
			ctx.Input.RunController = rule.controller
			ctx.Input.RunMethod = rule.method		
			break				
		}
	}
}

beego.InsertFilter("/*",beego.BeforeRouter,UrlManager)
```
