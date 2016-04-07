---
name: 请求数据处理
sort: 4
---

# 获取参数

我们经常需要获取用户传递的数据，包括 Get、POST 等方式的请求，beego 里面会自动解析这些数据，你可以通过如下方式获取数据：

- GetString(key string) string
- GetStrings(key string) []string
- GetInt(key string) (int64, error)
- GetBool(key string) (bool, error)
- GetFloat(key string) (float64, error)

使用例子如下：

```go
func (this *MainController) Post() {
	jsoninfo := this.GetString("jsoninfo")
	if jsoninfo == "" {
		this.Ctx.WriteString("jsoninfo is empty")
		return
	}
}
```

如果你需要的数据可能是其他类型的，例如是 int 类型而不是 int64，那么你需要这样处理：

```go
func (this *MainController) Post() {
	id := this.Input().Get("id")
	intid, err := strconv.Atoi(id)
}
```

更多其他的 request 的信息，用户可以通过 `this.Ctx.Request` 获取信息，关于该对象的属性和方法参考手册[Request](http://gowalker.org/net/http#Request)。

## 直接解析到 struct

如果要把表单里的内容赋值到一个 struct 里，除了用上面的方法一个一个获取再赋值外，beego 提供了通过另外一个更便捷的方式，就是通过 struct 的字段名或 tag 与表单字段对应直接解析到 struct。

定义struct：

```go
type user struct {
	Id    int         `form:"-"`
	Name  interface{} `form:"username"`
	Age   int         `form:"age"`
	Email string
}
```

表单：

	<form id="user">
		名字：<input name="username" type="text" />
		年龄：<input name="age" type="text" />
		邮箱：<input name="Email" type="text" />
		<input type="submit" value="提交" />
	</form>

Controller 里解析：

```go
func (this *MainController) Post() {
	u := user{}
	if err := this.ParseForm(&u); err != nil {
		//handle error
	}
}
```

注意：

* StructTag form 的定义和 [renderform方法](../view/view.md#renderform-%E4%BD%BF%E7%94%A8) 共用一个标签
* 定义 struct 时，字段名后如果有 form 这个 tag，则会以把 form 表单里的 name 和 tag 的名称一样的字段赋值给这个字段，否则就会把 form 表单里与字段名一样的表单内容赋值给这个字段。如上面例子中，会把表单中的 username 和 age 分别赋值给 user 里的 Name 和 Age 字段，而 Email 里的内容则会赋给 Email 这个字段。
* 调用 Controller ParseForm 这个方法的时候，传入的参数必须为一个 struct 的指针，否则对 struct 的赋值不会成功并返回 `xx must be  a struct pointer` 的错误。
* 如果要忽略一个字段，有两种办法，一是：字段名小写开头，二是：`form` 标签的值设置为 `-`

## 获取 Request Body 里的内容

在 API 的开发中，我们经常会用到 `JSON` 或 `XML` 来作为数据交互的格式，如何在 beego 中获取 Request Body 里的 JSON 或 XML 的数据呢？

1. 在配置文件里设置 `copyrequestbody = true`
2. 在 Controller 中

```go
func (this *ObjectController) Post() {
	var ob models.Object
	json.Unmarshal(this.Ctx.Input.RequestBody, &ob)
	objectid := models.AddOne(ob)
	this.Data["json"] = "{\"ObjectId\":\"" + objectid + "\"}"
	this.ServeJSON()
}
```

## 文件上传

在 beego 中你可以很容易的处理文件上传，就是别忘记在你的 form 表单中增加这个属性 `enctype="multipart/form-data"`，否则你的浏览器不会传输你的上传文件。

文件上传之后一般是放在系统的内存里面，如果文件的 size 大于设置的缓存内存大小，那么就放在临时文件中，默认的缓存内存是 64M，你可以通过如下来调整这个缓存内存大小:

	beego.MaxMemory = 1<<22

或者在配置文件中通过如下设置：

	maxmemory = 1<<22

Beego 提供了两个很方便的方法来处理文件上传：

- GetFile(key string) (multipart.File, *multipart.FileHeader, error)

	该方法主要用于用户读取表单中的文件名 `the_file`，然后返回相应的信息，用户根据这些变量来处理文件上传：过滤、保存文件等。

- SaveToFile(fromfile, tofile string) error

	该方法是在 GetFile 的基础上实现了快速保存的功能 
	fromfile 是提交时候的html表单中的name
	
```html
<form enctype="multipart/form-data" method="post">
	<input type="file" name="uploadname" />
	<input type="submit">
</form>
```

保存的代码例子如下：

```go
func (c *FormController) Post() {
	f, h, err := c.GetFile("uploadname")
	defer f.Close()
	if err != nil {
		fmt.Println("getfile err ", err)
	} else {
		c.SaveToFile("uploadname", "/www/"+h.Filename)
	}
}
```

## 数据绑定

支持从用户请求中直接数据bind到指定的对象,例如请求地址如下

	?id=123&isok=true&ft=1.2&ol[0]=1&ol[1]=2&ul[]=str&ul[]=array&user.Name=astaxie

```		
var id int  
this.Ctx.Input.Bind(&id, "id")  //id ==123

var isok bool  
this.Ctx.Input.Bind(&isok, "isok")  //isok ==true

var ft float64  
this.Ctx.Input.Bind(&ft, "ft")  //ft ==1.2

ol := make([]int, 0, 2)  
this.Ctx.Input.Bind(&ol, "ol")  //ol ==[1 2]

ul := make([]string, 0, 2)  
this.Ctx.Input.Bind(&ul, "ul")  //ul ==[str array]

user struct{Name}  
this.Ctx.Input.Bind(&user, "user")  //user =={Name:"astaxie"}
```
