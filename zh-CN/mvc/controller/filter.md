---
name: 过滤器
sort: 5
---

# 过滤器

beego 支持自定义过滤中间件，例如安全验证，强制跳转等。

过滤器函数如下所示：

	web.InsertFilter(pattern string, pos int, filter FilterFunc, opts ...FilterOpt)

InsertFilter 函数的三个必填参数，一个可选参数

- pattern 路由规则，可以根据一定的规则进行路由，如果你全匹配可以用 `*`
- position 执行 Filter 的地方，五个固定参数如下，分别表示不同的执行过程
  - BeforeStatic 静态地址之前
	- BeforeRouter 寻找路由之前
	- BeforeExec 找到路由之后，开始执行相应的 Controller 之前
	- AfterExec 执行完 Controller 逻辑之后执行的过滤器
	- FinishRouter 执行完逻辑之后执行的过滤器
- filter filter 函数 type FilterFunc func(*context.Context)
- opts
  1. web.WithReturnOnOutput: 设置 returnOnOutput 的值(默认 true), 如果在进行到此过滤之前已经有输出，是否不再继续执行此过滤器,默认设置为如果前面已有输出(参数为true)，则不再执行此过滤器
  2. web.WithResetParams: 是否重置 filters 的参数，默认是 false，因为在 filters 的 pattern 和本身的路由的 pattern 冲突的时候，可以把 filters 的参数重置，这样可以保证在后续的逻辑中获取到正确的参数，例如设置了 `/api/*` 的 filter，同时又设置了 `/api/docs/*` 的 router，那么在访问 `/api/docs/swagger/abc.js` 的时候，在执行 filters 的时候设置 `:splat` 参数为 `docs/swagger/abc.js`，但是如果不清楚 filter 的这个路由参数，就会在执行路由逻辑的时候保持 `docs/swagger/abc.js`，如果设置了 true，就会重置 `:splat` 参数.
  3. web.WithCaseSensitive: 是否大小写敏感。


如下例子所示，验证用户是否已经登录，应用于全部的请求：

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok && ctx.Request.RequestURI != "/login" {
        ctx.Redirect(302, "/login")
    }
}

web.InsertFilter("/*", web.BeforeRouter, FilterUser)
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
web.InsertFilter("/user/:id([0-9]+)", web.BeforeRouter, FilterUser)
```
## 过滤器实现路由

beego1.1.2 开始 Context.Input 中增加了 RunController 和 RunMethod, 这样我们就可以在执行路由查找之前,在 filter 中实现自己的路由规则.

如下示例实现了如何实现自己的路由规则:

```go
var UrlManager = func(ctx *context.Context) {
    // 数据库读取全部的 url mapping 数据
	urlMapping := model.GetUrlMapping()
	for baseurl,rule:=range urlMapping {
		if baseurl == ctx.Request.RequestURI {
			ctx.Input.RunController = rule.controller
			ctx.Input.RunMethod = rule.method
			break
		}
	}
}

web.InsertFilter("/*", web.BeforeRouter, web.UrlManager)
```

## Filter和FilterChain

在 v1.x 的设计中，Filter 并不能直接调用下一个 Filter。这导致了我们无法解决一个问题，即我们希望这个 Filter 能够在代码执行前后都执行一段逻辑。

例如，在考虑接入`Opentracing`和`prometheus`的时候，我们就遇到了这种问题。

考虑到这是一个通用的场景，我们在已有 Filter 的基础上，支持了`Filter-Chain`设计模式。

```go
type FilterChain func(next FilterFunc) FilterFunc
```

例如一个非常简单的例子：

```go
package main

import (
	"github.com/beego/beego/v2/core/logs"
	"github.com/beego/beego/v2/server/web"
	"github.com/beego/beego/v2/server/web/context"
)

func main() {
	web.InsertFilterChain("/*", func(next web.FilterFunc) web.FilterFunc {
		return func(ctx *context.Context) {
			// do something
			logs.Info("hello")
			// don't forget this
			next(ctx)

			// do something
		}
	})
}
```

这个例子里面，我们只是输出了一句"hello"，就调用了下一个 Filter。

在执行完`next(ctx)`之后，实际上，如果后面的 Filter 没有中断整个流程，那么这时候`OutPut`对象已经被赋值了，意味着能够拿到响应码等数据。

### Prometheus例子

```go
package main

import (
	"time"

	"github.com/beego/beego/v2/server/web"
	"github.com/beego/beego/v2/server/web/filter/prometheus"
)

func main() {
	// we start admin service
	// Prometheus will fetch metrics data from admin service's port
	web.BConfig.Listen.EnableAdmin = true

	web.BConfig.AppName = "my app"

	ctrl := &MainController{}
	web.Router("/hello", ctrl, "get:Hello")
	fb := &prometheus.FilterChainBuilder{}
	web.InsertFilterChain("/*", fb.FilterChain)
	web.Run(":8080")
	// after you start the server
	// and GET http://localhost:8080/hello
	// access http://localhost:8088/metrics
	// you can see something looks like:
	// http_request_web_sum{appname="my app",duration="1002",env="prod",method="GET",pattern="/hello",server="webServer:1.12.1",status="200"} 1002
	// http_request_web_count{appname="my app",duration="1002",env="prod",method="GET",pattern="/hello",server="webServer:1.12.1",status="200"} 1
	// http_request_web_sum{appname="my app",duration="1004",env="prod",method="GET",pattern="/hello",server="webServer:1.12.1",status="200"} 1004
	// http_request_web_count{appname="my app",duration="1004",env="prod",method="GET",pattern="/hello",server="webServer:1.12.1",status="200"} 1
}

type MainController struct {
	web.Controller
}

func (ctrl *MainController) Hello() {
	time.Sleep(time.Second)
	ctrl.Ctx.ResponseWriter.Write([]byte("Hello, world"))
}
```

别忘记了开启`prometheus`的端口。在你没有启动`admin`服务的时候，需要自己手动开启。

### Opentracing例子

```go
package main

import (
	"time"

	"github.com/beego/beego/v2/server/web"
	"github.com/beego/beego/v2/server/web/filter/opentracing"
)

func main() {
	// don't forget this to inject the opentracing API's implementation
	// opentracing2.SetGlobalTracer()

	web.BConfig.AppName = "my app"

	ctrl := &MainController{}
	web.Router("/hello", ctrl, "get:Hello")
	fb := &opentracing.FilterChainBuilder{}
	web.InsertFilterChain("/*", fb.FilterChain)
	web.Run(":8080")
	// after you start the server
}

type MainController struct {
	web.Controller
}

func (ctrl *MainController) Hello() {
	time.Sleep(time.Second)
	ctrl.Ctx.ResponseWriter.Write([]byte("Hello, world"))
}

```

别忘了调用`opentracing`库的`SetGlobalTracer`方法，注入真正的`opentracing API`的实现。
