---
name: 验证码
sort: 1
---

# 验证码

验证码分成两部分：一个是展示验证码的页面；一个是验证验证码。

于是我们可以先定义两个接口：
```go
// Get - address: http://127.0.0.1:8080/
func (ctrl *Controller) Get() {
	ctrl.TplName = "captcha.html"
	ctrl.Data["name"] = "Home"

	// don't forget this
	_ = ctrl.Render()
}

// Captcha - address: http://127.0.0.1:8080/sendCaptcha
func (ctrl *Controller) Captcha() {
	ctrl.TplName = "captcha.html"

	if !cpt.VerifyReq(ctrl.Ctx.Request) {
		logs.Error("Captcha does not match")
		_ = ctrl.Render()
		return
	}

	logs.Info("matched")
	_ = ctrl.Render()
}
```

前端页面可以很简单：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>

<body>
  Hello, world! This is {{.name}}, don't forget to remove the comments to test the form
{{/*  <form action="/sendCaptcha" method="post">*/}}
{{/*    {{create_captcha}}*/}}
{{/*    <input name="captcha" type="text">*/}}
{{/*  </form>*/}}
</body>
</html>
```
