# 参数解析

我们经常需要获取用户传递的数据，包括 Get、POST 等方式的请求，Beego 里面会自动解析这些数据，你可以通过如下方式获取数据：

- GetString(key string) string
- GetInt(key string) (int64, error)
- GetBool(key string) (bool, error)

使用例子如下：

	func (this *MainController) Post() {
		jsoninfo := this.GetString("jsoninfo")
		if jsoninfo == "" {
			this.Ctx.WriteString("jsoninfo is empty")
			return
		}
	}

如果你需要的数据可能是其他类型的，例如是 int 类型而不是 int64，那么你需要这样处理：

	func (this *MainController) Post() {
		id := this.Input().Get("id")
		intid, err := strconv.Atoi(id)
	}

更多其他的 request 的信息，用户可以通过 `this.Ctx.Request` 获取信息，关于该对象的属性和方法参考手册[Request](http://gowalker.org/net/http#Request)。