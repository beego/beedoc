---
name: Filters
sort: 2
---

# Filters

Beego supports custom filter middlewares. E.g.: user authentication and force redirecton.

Here is a filter function:

	beego.AddFilter(pattern, action string, filter FilterFunc)

AddFilter's three params:

- pattern: router rules. It can router based on the rules. Use `*` to match all.
- action: the place to execute Filter. There are four fixed params. They represent different execution process.
  - BeforeRouter: before finding router.
	- AfterStatic: after static rendering.
	- BeforeExec: After finding router and before executing the matched Controller.
	- AfterExec: After executing Controller.
- filter: filter function type FilterFunc func(*context.Context)


Here is the example to authenticate if the user is logged in for all requests:

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}

beego.AddFilter("*","AfterStatic",FilterUser)
```

>>>One thing you need to care about is he Filter useing session must use after `AfterStatic` because session is not initialized before it.


You can run the Filters by matching regex router rules:

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}
beego.AddFilter("/user/:id([0-9]+)","AfterStatic",FilterUser)
```
