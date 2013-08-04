# 过滤器

Beego 支持自定义过滤中间件，例如安全验证，强制跳转等。

如下例子所示，验证用户名是否是 admin，应用于全部的请求：

	var FilterUser = func(w http.ResponseWriter, r *http.Request) {
		if r.URL.User == nil || r.URL.User.Username() != "admin" {
			http.Error(w, "", http.StatusUnauthorized)
		}
	}
	
	beego.Filter(FilterUser)

还可以通过参数进行过滤，如果匹配参数就执行：

	beego.Router("/:id([0-9]+)", &admin.EditController{})
	beego.FilterParam("id", func(rw http.ResponseWriter, r *http.Request) {
		dosomething()
	})

当然你还可以通过前缀过滤：

	beego.FilterPrefixPath("/admin", func(rw http.ResponseWriter, r *http.Request) {
		dosomething()
	})