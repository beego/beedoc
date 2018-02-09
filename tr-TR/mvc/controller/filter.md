---
name: Filters
sort: 5
---

# Filters
Beego supports custom filter middlewares. E.g.: user authentication and force redirection.

# Activating Filters
Before filters can be used, filters must be activated.

Filters can be activated at the code level:

`beego.BConfig.WebConfig.Session.SessionOn = true`

Filters can also be activated in the configuration file:

```SessionOn = true```

Attempting to use a filter without activation will cause a `Handler crashed with error runtime error: invalid memory address or nil pointer dereference` error

# Inserting Filters

A filter function can be inserted as follows:

```go
beego.InsertFilter(pattern string, pos int, filter FilterFunc, params ...bool)
```

This is the FilterFunc signature:

```go
type FilterFunc func(*context.Context)
```

The *context* must be imported if this has not already been done:

```go
import "github.com/astaxie/beego/context"
```

InsertFilter's four parameters:

- `pattern`: string or regex to match against router rules. Use `/*` to match all.
- `pos`: the place to execute the Filter. There are five fixed parameters representing different execution processes.
	- beego.BeforeStatic: Before finding the static file.  
 	- beego.BeforeRouter: Before finding router.
	- beego.BeforeExec: After finding router and before executing the matched Controller.
	- beego.AfterExec: After executing Controller.
	- beego.FinishRouter: After finishing router.
- `filter`: filter function type FilterFunc func(*context.Context)
- `params[0]` - skip: whether to continue running if has output. default is false.
- `params[1]` - reset params: whether to reset parameters to their previous values after the filter has completed.

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

>Filters which use session must be executed after `BeforeRouter` because session is not initialized before that. Beego session module must be enabled first. (see [Session control](../controller/session.md))


Filters can be run against requests which use a regex router rule for matching:

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
Context.Input has new features `RunController` and `RunMethod` from beego version 1.1.2.  These can control the router in the filter and skip the Beego router rule.

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
