---
name: 验证码
sort: 1
---

# 验证码

验证码分成两部分：一个是展示验证码的页面；一个是验证验证码。

于是我们可以先定义两个接口：
```
package controllers

import (
	web "github.com/beego/beego/v2/server/web"
	"github.com/beego/beego/v2/client/cache"
	"github.com/beego/beego/v2/server/web/captcha"
)

var cpt *captcha.Captcha

func init() {
	// use beego cache system store the captcha data
	store := cache.NewMemoryCache()
	cpt = captcha.NewWithFilter("/captcha/", store)
}

type MainController struct {
	web.Controller
}

func (this *MainController) Get() {
	this.TplName = "index.tpl"
}

func (this *MainController) Post() {
	this.TplName = "index.tpl"

	this.Data["Success"] = cpt.VerifyReq(this.Ctx.Request)
}
```

前端页面可以很简单：

```html
{{.Success}}
<form action="/" method="post">
	{{create_captcha}}
	<input name="captcha" type="text">
</form>
```
