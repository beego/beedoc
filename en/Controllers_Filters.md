# Filters

Beego supports customized filter and middleware, such as security verification, force redirect, etc.

Here is an example of verify user name of all requests, check if it's admin.

	var FilterUser = func(w http.ResponseWriter, r *http.Request) {
		if r.URL.User == nil || r.URL.User.Username() != "admin" {
			http.Error(w, "", http.StatusUnauthorized)
		}
	}
	
	beego.Filter(FilterUser)

You can also filter by arguments:

	beego.Router("/:id([0-9]+)", &admin.EditController{})
	beego.FilterParam("id", func(rw http.ResponseWriter, r *http.Request) {
		dosomething()
	})

Filter by prefix is also available:

	beego.FilterPrefixPath("/admin", func(rw http.ResponseWriter, r *http.Request) {
		dosomething()
	})