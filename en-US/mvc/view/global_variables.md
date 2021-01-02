---
name: Pagination
sort: 5
---

# Global Variables

Variables common to all requests can be added using a base controller using the `Prepare` method.

Usage examples include the username of the current user for display in a toolbar or the request URL for highlighting active menu items. 

First, create a common/base controller in the controllers package:
```
app_root
├── controllers
│   ├── base.go <-- base controller
│   └── default.go
└── main.go
```

Your base controller should embed the `web.Controller` from `github.com/beego/beego/v2/server/web`. From here, the `Prepare` method should be defined, containing any logic required for global variables:
```go
// app_root/controllers/base.go
package controllers

import (
	"github.com/beego/beego/v2/server/web"
)

type BaseController struct {
	web.Controller
}

// Runs after Init before request function execution
func (c *BaseController) Prepare() {
	c.Data["RequestUrl"] = c.Ctx.Input.URL()
}

// Runs after request function execution
func (c *BaseController) Finish() {
	// Any cleanup logic common to all requests goes here. Logging or metrics, for example.
}
```

All other controllers should embed `BaseController` instead of `web.Controller`:
```go
// app_root/controllers/default.go
package controllers

type DefaultController struct {
	BaseController
}

func (c *DefaultController) Index() {
	// your controller logic
}
```

From here your views can access these global variables, in both individual templates and all parent templates:
```gotemplate
<div>{{ $.RequestUrl }}</div>
```
