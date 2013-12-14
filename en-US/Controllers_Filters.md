# Filters

Beego supports customized filters and middleware, such as security verification, forced redirect, etc.

Function call will be like:

	beego.AddFilter(pattern, action string, filter FilterFunc)

Three arguments:

- pattern: router rules, according to your rules to route, and use `*` for full match.
- action: the place to execute Filter, there are 4 fixed arguments where each one represents different execute process.
	- BeforeRouter: before routing
	- AfterStatic: after static rendered
	- BeforeExec: after found  route, before executing controller logic
	- AfterExec: after executed controller logic
- filter: filter function, type FilterFunc func(*context.Context)

The following example can be used to verify user log in and apply to all requests:

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}

beego.AddFilter("*","BeforRouter",FilterUser)
```

Or use regexp to filter:

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}
beego.AddFilter("/user/:id([0-9]+)","BeforRouter",FilterUser)
```