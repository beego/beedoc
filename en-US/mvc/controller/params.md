---
name: Request parameters
sort: 4
---

# Accept parameters

We need to access the data passed by user from GET, POST and other methods. Beego will parse this data automatically. You can get the data by:

- GetString(key string) string
- GetStrings(key string) []string
- GetInt(key string) (int64, error)
- GetBool(key string) (bool, error)
- GetFloat(key string) (float64, error)

For example:

```go
func (this *MainController) Post() {
	jsoninfo := this.GetString("jsoninfo")
	if jsoninfo == "" {
		this.Ctx.WriteString("jsoninfo is empty")
		return
	}
}
```

If you need the data in other types. Such as int other than int64, then you need to do so:

```go
func (this *MainController) Post() {
	id := this.Input().Get("id")
	intid, err := strconv.Atoi(id)
}
```

For more request information, you can get by `this.Ctx.Request`. For more reference see [Request](http://gowalker.org/net/http#Request).

## Parse to struct

If you want to assign the data in a form into struct, except assign them one by one, Beego have a easier way to do that which is mapping struct key with form name and parse all parameters into struct.

Define struct:

```go
type user struct {
	Id    int         `form:"-"`
	Name  interface{} `form:"username"`
	Age   int         `form:"age"`
	Email string
}
```

Define form:

	<form id="user">
		name：<input name="username" type="text" />
		age：<input name="age" type="text" />
		email：<input name="Email" type="text" />
		<input type="submit" value="submit" />
	</form>

Parsing in Controller:

```go
func (this *MainController) Post() {
	u := user{}
	if err := this.ParseForm(&u); err != nil {
		//handle error
	}
}
```

Notes:

* The definition of structTag form and [renderform method](../view/view.md#renderform) are using the same tag.
* While define struct, if there is the form tag after that key, it will assign the value in the form which has the same name as that tag, otherwise it will assign the value in the form which has the same name as that field key. In the above example, Form value username and age will be assigned to Name and Age in user strut and Email will be assigned to Email in struct.
* While call method ParseForm of Controller, the parameter pass in must be a pointer of struct, otherwise the assignment won't success and it will return a `xx must be a struct pointer` error.
* If you want to ignore some fields, there are two ways: one is using lowercase for that field; another is use `-` as the value of the tag.

## Retrive the data from request body

In API application development, we we always use `JSON` or `XML` as the data type. So how can we retrive the data from the request body?

1. Set `copyrequestbody = true` in configuration file.
2. Then in Controller you can

```go
func (this *ObejctController) Post() {
	var ob models.Object
	json.Unmarshal(this.Ctx.Input.RequestBody, &ob)
	objectid := models.AddOne(ob)
	this.Data["json"] = "{\"ObjectId\":\"" + objectid + "\"}"
	this.ServeJson()
}
```

## Uploading files

In Beego, you can uploading files easily. Just remember set attribute `enctype="multipart/form-data"` in your form, otherwise your browser won't upload your file.

Usually the uploaded file is stored in the system memory, but if the file size it bigger than the memory size limitation in the configuration file, the file will be stored in a temporaty file. The default memory size is 64M and you can change it by:

	beego.MaxMemory = 1<<22

Or you can set it in the configuration file:

	maxmemory = 1<<22

Beego provided two functions to handle file upload easily:

- GetFile(key string) (multipart.File, *multipart.FileHeader, error)

This method is used to read the file name `the_file` from form and return the information then you can process the uploaded file based on the info, such as filter or save the file.

- SaveToFile(fromfile, tofile string) error

This method impelemented the saving function based on the method `GetFile`

Here is the example of saving file:

```go
func (this *MainController) Post() {
	this.SaveToFile("the_file","/var/www/uploads/uploaded_file.txt")
}
```
## Data Bind

Data bind let user bind the request data to a params, the request url as follow:

	?id=123&isok=true&ft=1.2&ol[0]=1&ol[1]=2&ul[]=str&ul[]=array&user.Name=astaxie

```		
var id int  
ctx.Input.Bind(&id, "id")  //id ==123

var isok bool  
ctx.Input.Bind(&isok, "isok")  //isok ==true

var ft float64  
ctx.Input.Bind(&ft, "ft")  //ft ==1.2

ol := make([]int, 0, 2)  
ctx.Input.Bind(&ol, "ol")  //ol ==[1 2]

ul := make([]string, 0, 2)  
ctx.Input.Bind(&ul, "ul")  //ul ==[str array]

user struct{Name}  
ctx.Input.Bind(&user, "user")  //user =={Name:"astaxie"}