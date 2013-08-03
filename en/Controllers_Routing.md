# Routing

### Default RESTful rules

The main function of router is to connect request URL and handler. Beego wrapped `Controller`, so it connects request URL and `ControllerInterface`. The `ControllerInterface` has following methods:

	type ControllerInterface interface {
		Init(ct *Context, cn string)
		Prepare()
		Get()
		Post()
		Delete()
		Put()
		Head()
		Patch()
		Options()
		Finish()
		Render() error
	}

`beego.Controller` implemented all of them, so you just use this struct as anonymous field in your controller struct. Of course you have to overload corresponding methods for more specific usages.

Users can use following ways to register route rules:

	beego.Router("/", &controllers.MainController{})
	beego.Router("/admin", &admin.UserController{})
	beego.Router("/admin/index", &admin.ArticleController{})
	beego.Router("/admin/addpkg", &admin.AddController{})

For more convenient configure route rules, Beego references the idea from sinatra, so it supports more kinds of route rules as follows:

- beego.Router("/api/:id([0-9]+)", &controllers.RController{})

		Customized regular expression match 	// match /api/123 :id= 123

- beego.Router("/news/:all", &controllers.RController{})

		Match rest of all // match /news/path/to/123.html :all= path/to/123.html

- beego.Router("/user/:username([\w]+)", &controllers.RController{})
 
		Regular expression // match /user/astaxie    :username = astaxie

- beego.Router("/download/`*`.`*`", &controllers.RController{})

		Wildcard character // match /download/file/api.xml     :path= file/api   :ext=xml

- beego.Router("/download/ceshi/`*`", &controllers.RController{})

		wildcard character match rest of all // match  /download/ceshi/file/api.json  :splat=file/api.json

- beego.Router("/:id:int", &controllers.RController{})
 
		Match type int  // match :id is int type, Beego uses regular expression ([0-9]+) automatically

- beego.Router("/:hi:string", &controllers.RController{})

		Match type string // match :hi is string type, Beego uses regular expression ([\w]+) automatically

### Customized methods and RESTful rules


