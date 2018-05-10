---
name: Request parameters
sort: 4
---

# Accept parameters

Beego will automatically parse data passed by user from GET, POST and other methods. This data can be accessed using:

- GetString(key string) string
- GetStrings(key string) []string
- GetInt(key string) (int, error)
- GetInt8(key string) (int8, error)
- GetInt16(key string) (int16, error)
- GetInt32(key string) (int32, error)
- GetInt64(key string) (int64, error)
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

More information about the request can be retrieved by accessing `this.Ctx.Request`. For more details see [Request](http://gowalker.org/net/http#Request).

## Parse to struct

Data submitted from a form may be assigned to a struct by mapping struct fields to the form's input elements and parsing all data into a struct.

Define struct:

```go
type User struct {
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
	u := User{}
	if err := this.ParseForm(&u); err != nil {
		//handle error
	}
}
```

Notes:

* The same tag is used for the definition of structTag form and [renderform method](../view/view.md#renderform).
* If there is a form tag after the key while defining the struct, the value in the form which has the same name as that tag will be assigned.  Otherwise, the value in the form which has the same name as that field name will be assigned. In the above example, Form values username and age will be assigned to Name and Age in user struct and Email will be assigned to Email in struct.
* While calling the method ParseForm of the Controller the parameter passed in must be a pointer to a struct. Otherwise, the assignment will fail and will return a `xx must be a struct pointer` error.
* Fields can be ignored by using lowercase for that field or by using `-` as the value of the tag.

## Automatic Parameter Routing

Automatic parameter routing removes the need for boilerplate code like `this.GetString(..)`, `this.GetInt(..)` etc.  Instead https parameters are injected directly as method parameters and the method return values are rendered as http responses. This works in conjunction with annotations to create a seamless integration.

### How does it work?

Start by defining a regular controller method with a `@router` annotation and add parameters to the method signature

```go
// @router /tasks
func (c *TaskController) MyMethod(id int) {
...
}
```

When an http request comes in that matches the defined routing Beego will scan the parameters in the method signature and try to find matching http request paramters, where method parameter name is the http request parameter name.  Beego will then convert them to the correct parameter type and pass them to your method. By default Beego will look for parameters in the quey string (when using `GET`) or form data (when using `POST`). If your routing definition contains parameters Beego will automatically search for them in the path:

```go
// @router /task/:id
func (c *TaskController) MyMethod(id int) {
...
}
```

Annotations can also be used to indicate a parameter is passed in a header or in the request body.  Bego will search for it accordingly.

If a parameter is not found in the http request it will be passed to your controller method as a zero value (i.e. 0 for int, false for bool etc.). If a default value for that parameter has been defined in annotations, Beego will pass that default value if it is missing. To differentiate between missing parameters and default values define the parameter as a pointer, e.g.: 

```go
// @router /tasks
func (c *TaskController) MyMethod(id *int) {
...
}
```
If the parameter in the above case was missing, `id` would be null.  If the parameter exists and equals to zero, `id` would be 0. When using annotations to create swagger documentation a parameter can be marked as `required`. If the parameter is missing in the request a `400 Bad Request` error will be returned to the client:

```go
// @Param   id     query   int true       "task id"
// @router /tasks
func (c *TaskController) MyMethod(id *int) {
...
}
```

If Beego can not convert the parameter to the requested type (i.e. if a string is passed that can not be parsed as an integer) an error will be returned to the client.

The following table shows which types are supported and how they are parsed:

Data Type | Location | Example | Comment
----------|----------|---------|----------
int, int64, uint etc. | anywhere | "1","-100" | Uses `strconv.Atoi(value)` 
float32,float64 | anywhere | "1.5", "-3.5" | Uses `strconv.ParseFloat()` 
bool | anywhere | "1", "T", "false" | Uses `strconv.ParseBool()` 
time.Time | anywhere | "2017-01-01" "2017-01-01T00:00:00Z" | Uses RFC3339 or short date format (`"2006-01-02"`) when parsing
[]string, []int etc. | query | "A,B,C" "1,2,3" | Any type is supported as a slice. When it is located in the query string, it is parsed as a comma separated list
[]string, []int etc. | body | \["A","B","C"] [1,2,3] | When slices are located in the request body they are parsed as JSON arrays
[]byte | anywhere | "ABC" | byte[] is not treated as an array but as a string
\*int, \*string, \*float etc. | anywhere | Pointers will receive null if the parameter is missing from the request otherwise, it will behave the same as defined in the other rows
structs, all others | anywhere | {"X":"Y"} | structs and other types (e.g. maps) are always parsed as JSON using `json.Unmarshal()`

### How are method return values handled?
Method return values are handled automatically in the same manner as parameters. A method can have one or more return values and Beego will render all of them to the response. The best practice is to define one result as a 'regular' type (i.e. a map, a struct or any other data type) and another as an error data type:

```go
// @Param   id     query   int true       "task id"
// @router /tasks
func (c *TaskController) MyMethod(id *int) (*MyModel, error) {
...
}
```

In the code above the method can return three different results:
- Only `MyModel` (nil `error`)
- Only `error` (nil `MyModel`)
- Both `MyModel` and `error`

When a regular type is returned it is rendered directly as JSON, and when an error is returned it is rendered as an http status code. Beego will handle all cases correctly and supports returning both response body and http error if both values are non-nil.

A few helper types will return common http status codes easily. For example, `404 Not Found`, `302 Redirect` or other http status codes like in the following example:

```go
func (c *TaskController) MyMethod(id *int) (*MyModel, error) {
  if /* not found */ {
    return nil, context.NotFound
  } else if /* some error */ {
    return nil, context.StatusCode(401)
  } else {
  	return &MyModel{}, nil
  }
}
```
### How annotations work in conjuction with method parameters?

Automatic Parameter Routing works best together with `@Param` annotations. The following features are supported with annotations:
- If a parameter is marked as required, Beego will return an error if the parameter is not present in the http request:  

```go
// @Param   brand_id    query   int true       "brand id"
```
(the `true` option in the annotation above indicates that brand_id is a required parameter)
- If a parameter has a default value and it does not exist in the http request, Beego will pass that default value to the method:  

```go
// @Param   brand_id    query   int false  5  "brand id"
```
(the `5` in the annotation above indicates that this is the default value for that parameter)
- The location parameter in the annotation indicates where beego will search for that parameter in the request (i.e. query, header, body etc.)  

```go
// @Param   brand_id    path   	int 	true  "brand id"
// @Param   category    query 	string	false "category" 
// @Param   token	header  string	false "auth token"
// @Param   task	body	{models.Task} false "the task object"
```
- If a parameter name in the http request is different from the method parameter name, you can "redirect" the parameter using the `=>` notation. This is useful, for example, When a header name is `X-Token` and the method parameter is named `x_token`:  

```go
// @Param   X-Token=>x_token	header  string	false "auth token"
```
- A parameter swagger data type can be inferred from the method to make maintainance easier. Use the `auto` data type and Beego will generate the correct swagger documentation:  

```go
// @Param   id     query   auto true       "task id"
// @router /tasks
func (c *TaskController) MyMethod(id int) (*MyModel, error) {
...
}
```

## Retrieving data from request body

In API application development always use `JSON` or `XML` as the data type. To retrieve the data from the request body:

1. Set `copyrequestbody = true` in configuration file.
2. Then in the Controller you can

```go
func (this *ObjectController) Post() {
	var ob models.Object
	json.Unmarshal(this.Ctx.Input.RequestBody, &ob)
	objectid := models.AddOne(ob)
	this.Data["json"] = map[string]interface{}{"ObjectId": objectid }
	this.ServeJSON()
}
```

## Uploading files

To upload files with Beego set attribute `enctype="multipart/form-data"` in your form.

Usually an uploaded file is stored in the system memory, but if the file size is larger than the memory size limitation in the configuration file, the file will be stored in a temporary file. The default memory size is 64M but can be changed using:

	beego.MaxMemory = 1<<22

Or it can be set manualy in the configuration file:

	maxmemory = 1<<22

Beego provides two functions to handle file uploads:

- GetFile(key string) (multipart.File, *multipart.FileHeader, error)

This method is used to read the file name `the_file` from form and return the information. The uploaded file can then be processed based on this information, such as filter or save the file.

- SaveToFile(fromfile, tofile string) error

This method implements the saving function based on the method `GetFile`

Here is an example of saving a file:

```go
func (this *MainController) Post() {
	this.SaveToFile("the_file","/var/www/uploads/uploaded_file.txt")
}
```
## Data Bind

Data bind lets the user bind the request data to a variable, the request url as follows:

	?id=123&isok=true&ft=1.2&ol[0]=1&ol[1]=2&ul[]=str&ul[]=array&user.Name=astaxie

```go
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
```
