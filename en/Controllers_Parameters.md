# Parameters

We always need to get data from users, including methods like GET, POST, etc. Beego parses this data automatically, and you can access them with following code:

- GetString(key string) string
- GetInt(key string) (int64, error)
- GetBool(key string) (bool, error)

Usage example:

```go
func (this *MainController) Post() {
	jsoninfo := this.GetString("jsoninfo")
	if jsoninfo == "" {
		this.Ctx.WriteString("jsoninfo is empty")
		return
	}
}
```

If you need other types that are not included above, like you need int64 instead of int, then you need to do following way:

```go
func (this *MainController) Post() {
	id := this.Input().Get("id")
	intid, err := strconv.Atoi(id)
}
```

To use `this.Ctx.Request` for more information about request, and object properties and method please read [Request](http://gowalker.org/net/http#Request).

## Parse to struct

If you want to parse form data to a struct, Beego provides you a convenient way to parse form fields according to struct fields or tags.

Define a struct:

```go
type user struct {
	Id    int
	Name  interface{} `form:"username"`
	Age   int         `form:"age"`
	Email string
}
```

Form:

	<form id="user">
		Name:<input name="username" type="text" />
		Age:<input name="age" type="text" />
		Email:<input name="Email" type="text" />
		<input type="submit" value="Submit" />
	</form>

Parse in controller:

```go
func (this *MainController) Post() {
	u := user{}
	if err := this.ParseForm(&u); err != nil {
		//handle error
	}
}
```

Attention:

* The relationship of struct field names and form field names follows the rules of JSON parsing, which means it uses struct field name to find corresponding form field unless you specified another name in fields tag. In above example, the values from the `username` and `age` form fields will be assigned to the `Name` and `Age` fields in the user struct, and value of `Email` will be assigned to `Email`.
* You have to pass struct pointer when you call the `ParseForm` method of the controller, otherwise it will fail to parse with error `xx must be a struct pointer`.

## Get content of request body

In the development of API applications, we often use JSON and XML as data exchange formats, but how do we access JSON or XML formatted data in beego request bodies?

1. Enable the `copyrequestbody = true` setting in configuration file.
2. Do following steps in controller:

```go
func (this *ObejctController) Post() {
	var ob models.Object
	json.Unmarshal(this.Ctx.Input.RequestBody, &ob)
	objectid := models.AddOne(ob)
	this.Data["json"] = "{\"ObjectId\":\"" + objectid + "\"}"
	this.ServeJson()
}
```
