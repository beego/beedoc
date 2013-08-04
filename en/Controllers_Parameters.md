# Parameters

We always need to get data from users, including methods like GET, POST, etc. Beego parses these data automatically, and you can access them by following code:

- GetString(key string) string
- GetInt(key string) (int64, error)
- GetBool(key string) (bool, error)

Usage example:

	func (this *MainController) Post() {
		jsoninfo := this.GetString("jsoninfo")
		if jsoninfo == "" {
			this.Ctx.WriteString("jsoninfo is empty")
			return
		}
	}

If you need other types that are not included above, like you need int64 instead of int, then you need to do following way:

	func (this *MainController) Post() {
		id := this.Input().Get("id")
		intid, err := strconv.Atoi(id)
	}

To use `this.Ctx.Request` for more information about request, and object properties and method please read [Request](http://gowalker.org/net/http#Request).