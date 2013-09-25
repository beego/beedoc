# Filters

Beego supports customized filters and middleware, such as security verification, forced redirect, etc.

Here is an example that verifies user name of all requests, checking if it's admin.

```go
var FilterUser = func(w http.ResponseWriter, r *http.Request) {
	if r.URL.User == nil || r.URL.User.Username() != "admin" {
		http.Error(w, "", http.StatusUnauthorized)
	}
}

beego.Filter(FilterUser)
```

You can also filter by arguments:

```go
beego.Router("/:id([0-9]+)", &admin.EditController{})
beego.FilterParam("id", func(rw http.ResponseWriter, r *http.Request) {
	dosomething()
})
```

Filter by prefix is also available:

```go
beego.FilterPrefixPath("/admin", func(rw http.ResponseWriter, r *http.Request) {
	dosomething()
})
```
