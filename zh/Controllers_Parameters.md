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

## 直接解析到 struct

如果要把表单里的内容赋值到一个 struct 里，除了用上面的方法一个一个获取再赋值外，beego提供了通过另外一个更便捷的方式，就是通过 struct 的字段名或 tag 与表单字段对应直接解析到 struct。

定义struct:

	type user struct {
		Id    int
		Name  interface{} `form:"username"`
		Age   int         `form:"age"`
		Email string
	}

表单：

	<form id="user">
		名字：<input name="username" type="text" />
		年龄：<input name="age" type="text" />
		邮箱：<input name="Email" type="text" />
		<input type="submit" value="提交" />
	</form>

Controller里解析：

	func (this *MainController) Post() {
		u := user{}
		if err := this.ParseForm(&u); err != nil {
			//handle error
		}
	}

注意：

* 定义struct时，字段名后如果有form这个tag，则会以把form表单里的name和tag的名称一样的字段赋值给这个字段，否则就会把form表单里与字段名一样的表单内容赋值给这个字段。如上面例子中，会把表单中的username和age分别赋值给user里的Name和Age字段，而Email里的内容则会赋给Email这个字段。
* 调用Controller ParseForm这个方法的时候，传入的参数必须为一个struct的指针，否则对struct的赋值不会成功并返回`xx must be  a struct pointer`的错误。

## 获取Request Body里的内容

在API的开发中，我们经常会用到json或xml来作为数据交互的格式，如何在beego中获取Request Body里的json或xml的数据呢？

1. 在配置文件里设置`copyrequestbody = true`
2. 在Controller中

	func (this *ObejctController) Post() {
		var ob models.Object
		json.Unmarshal(this.Ctx.RequestBody, &ob)
		objectid := models.AddOne(ob)
		this.Data["json"] = "{\"ObjectId\":\"" + objectid + "\"}"
		this.ServeJson()
	}

