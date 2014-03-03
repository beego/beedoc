---
name: Filters
sort: 2
---

# Filters

Beego supports custom filter middlewares. E.g.: user authentication and force redirecton.

Here is a filter function:

	beego.InsertFilter(pattern string, pos int, filter FilterFunc)

InsertFilter's three params:

- pattern: router rules. It can router based on the rules. Use `*` to match all.
- pos: the place to execute Filter. There are five fixed params. They represent different execution process.
 	- beego.BeforeRouter: before finding router.
	- beego.AfterStatic: after static rendering.
	- beego.BeforeExec: After finding router and before executing the matched Controller.
	- beego.AfterExec: After executing Controller.
	- beego.FinishRouter: After finishing router.
- filter: filter function type FilterFunc func(*context.Context)


Here is the example to authenticate if the user is logged in for all requests:

```go
var FilterUser = func(ctx *context.Context) {
    _, ok := ctx.Input.Session("uid").(int)
    if !ok {
        ctx.Redirect(302, "/login")
    }
}

beego.InsertFilter("*", beego.AfterStatic, FilterUser)
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
beego.InsertFilter("/user/:id([0-9]+)", beego.AfterStatic, FilterUser)
```
