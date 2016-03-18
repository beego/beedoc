---
name: Filters
sort: 5
---

# Filters

Beego supports custom filter middlewares. E.g.: user authentication and force redirection.

To use a filter function, insert it as follows:

```go
beego.InsertFilter(pattern string, pos int, filter FilterFunc, skip ...bool)
```

Here is the FilterFunc signature:

```go
type FilterFunc func(*context.Context)
```

Don't forget to import *context* if it hasn't been imported yet:

```go
import "github.com/astaxie/beego/context"
```

InsertFilter's four parameters:

- pattern: string or regex to match against router rules. Use `/*` to match all.
- pos: the place to execute the Filter. There are four fixed parameters. They represent different execution processes.
 	- beego.BeforeRouter: before finding router.
	- beego.BeforeExec: After finding router and before executing the matched Controller.
	- beego.AfterExec: After executing Controller.
	- beego.FinishRouter: After finishing router.
- filter: filter function type FilterFunc func(*context.Context)
- skip: whether to continue running if has output. default is false.

> from beego version 1.3 AddFilter has been removed

Here is an example to authenticate if the user is logged in for all requests:

```go
var FilterUser = func(ctx *context.Context) {
    if strings.HasPrefix(ctx.Input.URL(), "/login") {
    	return
    }
    
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}

beego.InsertFilter("/*", beego.BeforeRouter, FilterUser)
```

>For filters which use session, they must be executed after `BeforeRouter` because session is not initialized before that, and don't forget to enable Beego session module (see [Session control](../controller/session.md))


You can run filters against requests which use a regex router rule for matching:

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}
beego.InsertFilter("/user/:id([0-9]+)", beego.BeforeRouter, FilterUser)
```
## Filter Implementation UrlManager
Context.Input has new features `RunController` and `RunMethod` from beego version 1.1.2. So we can control the router in our filter and skip the Beego's router rule.

For example:

```go
var UrlManager = func(ctx *context.Context) {
    // read urlMapping data from database
	urlMapping := model.GetUrlMapping()
	for baseurl,rule := range urlMapping {
		if baseurl == ctx.Request.RequestURI {
			ctx.Input.RunController = rule.controller
			ctx.Input.RunMethod = rule.method
			break
		}
	}
}

beego.InsertFilter("/*", beego.BeforeRouter, UrlManager)
```
