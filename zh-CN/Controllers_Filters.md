# 过滤器

beego 支持自定义过滤中间件，例如安全验证，强制跳转等。

过滤器函数如下所示：

	beego.AddFilter(pattern, action string, filter FilterFunc)

AddFilter函数的三个参数

- pattern 路由规则，可以根据一定的规则进行路由，如果你全匹配可以用 `*`
- action 执行 Filter 的地方，四个固定参数如下，分别表示不同的执行过程
	- BeforRouter 寻找路由之前
	- AfterStatic 静态渲染之后
	- BeforExec 找到路由之后，开始执行相应的 Controller 之前
	- AfterExec 执行完 Controller 逻辑之后执行的过滤器
- filter filter 函数 type FilterFunc func(*context.Context)

如下例子所示，验证用户是否已经登录，应用于全部的请求：

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}

beego.AddFilter("*","AfterStatic",FilterUser)
```

还可以通过正则路由进行过滤，如果匹配参数就执行：

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}
beego.AddFilter("/user/:id([0-9]+)","AfterStatic",FilterUser)
```
