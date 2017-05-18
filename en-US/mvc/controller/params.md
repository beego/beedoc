---
name: Request parameters
sort: 4
---

# Accept parameters

We need to access the data passed by user from GET, POST and other methods. Beego will parse this data automatically. You can get the data by:

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

For more information about the request, you can get this by accessing `this.Ctx.Request`. For more details see [Request](http://gowalker.org/net/http#Request).

## Parse to struct

If you wish to assign the data submitted from a form into a struct rather than assigning them one by one, Beego has an easier way to do that. This approach consists of mapping struct fields to the  form's input elements and parsing all data into a struct.

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

* The definition of structTag form and [renderform method](../view/view.md#renderform) are using the same tag.
* While defining the struct, if there is a form tag after the key, it will assign the value in the form which has the same name as that tag, otherwise it will assign the value in the form which has the same name as that field name. In the above example, Form value username and age will be assigned to Name and Age in user struct and Email will be assigned to Email in struct.
* While calling the method ParseForm of the Controller, the parameter passed in must be a pointer to a struct, otherwise the assignment will fail and it will return a `xx must be a struct pointer` error.
* If you want to ignore some fields, there are two ways: the first is using lowercase for that field; the second is to use `-` as the value of the tag.

## Automatic Parameter Routing

Automatic parameter routing removes the need for boilerplate code like `this.GetString(..)`, `this.GetInt(..)` etc. and instead injects https parameters directly as method parameters and renders the method return values as http response. This works in conjunction with annotations to create a seamless integration.

### How does it work?

Start by defining a regular controller method with a `@router` annotation and add parameters to the method signature
```go
// @router /tasks
func (c *TaskController) MyMethod(id int) {
...
}
```

When an http request comes in that matches the routing you defined, Beego will scan the parameters in the method signature and will try to find matching http request paramters (where method parameter name is the http request parameter name), convert them to the correct parameter type and pass them to your method. By default, Beego will look for parameters in the quey string (when using `GET`) or form data (when using `POST`). If your routing definition contains parameters, then Beego will automatically search for them in the path:
```go
// @router /task/:id
func (c *TaskController) MyMethod(id int) {
...
}
```

You can also use annotations to indicate a parameter is passed in a header or in the request body and bego will search for it accordingly.
If a parameter is not found in the http request, it will be passed to your controller method as a zero value (i.e. 0 for int, false for bool etc.). If you define a default value for that parameter in annotations, beego will pass that default value if it is missing. If you want to differentiate between missing parameters and default values, you can define the parameter as a pointer, e.g.: 
```go
// @router /tasks
func (c *TaskController) MyMethod(id *int) {
...
}
```
In the above case, if the paramter was missing, `id` will be null but if it exists and equals to zero, `id` will be 0. If you are using annotations to create swagger documentation, you can mark a parameter as `required`. In this case, if the parameter is missing in the request, a `400 Bad Request` error will be returned to the client:
```go
// @Param   id     query   int true       "task id"
// @router /tasks
func (c *TaskController) MyMethod(id *int) {
...
}
```

Also, If a Beego can not convert the parameter to the requested type (i.e. if you pass a string that can not be parsed as an integer), an error will be returned to the client.

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
The same way parameters are handled automatically, a method return value is also handled automatically. A method can have one or more return values and Beego will render all of them to the response. The best practice is to define one result as a 'regular' type (i.e. a map, a struct or any other data type) and another error data type (Same as the best practice for regular go methods):

```go
// @Param   id     query   int true       "task id"
// @router /tasks
func (c *TaskController) MyMethod(id *int) (*MyModel, error) {
...
}
```

In the code above, the method can return three different results:
- Only `MyModel` (nil `error`)
- Only `error` (nil `MyModel`)
- Both `MyModel` and `error`

When a regular type is returned, it is rendered to the response directly as JSON. When an error is returned it is rendered as an http status code. Altough the first two options are more common, Beego will handle all cases correctrly and supports returning both response body and http error if both values are non-nil.
A few helper types allow you toy return common http status codes easily. For example, you can return `404 Not Found`, `302 Redirect` or other http status codes like in the following example:
```go
func (c *TaskController) MyMethod(id *int) (*MyModel, error) {
  if /* not found */ {
    return nil, context.NotFound
  } else if /* some error */ {
    return nil, context.StatusCode(401)
  } else /* redirect */ {
  	return nil, context.Redirect("/login")
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
- A parameter swagger data type can be inferred from the method to make maintainance easier. Just use the `auto` data type and let Bee generate the correct swagger documentation:  
```go
// @Param   id     query   auto true       "task id"
// @router /tasks
func (c *TaskController) MyMethod(id int) (*MyModel, error) {
...
}
```

## Retrieving data from request body

In API application development, we always use `JSON` or `XML` as the data type. So how can we retrieve the data from the request body?

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

In Beego, you can upload files easily. Just remember to set attribute `enctype="multipart/form-data"` in your form, otherwise your browser won't upload your file.

Usually an uploaded file is stored in the system memory, but if the file size is bigger than the memory size limitation in the configuration file, the file will be stored in a temporary file. The default memory size is 64M and you can change it by:

	beego.MaxMemory = 1<<22

Or you can set it in the configuration file:

	maxmemory = 1<<22

Beego provides two functions to handle file uploads easily:

- GetFile(key string) (multipart.File, *multipart.FileHeader, error)

This method is used to read the file name `the_file` from form and return the information then you can process the uploaded file based on the info, such as filter or save the file.

- SaveToFile(fromfile, tofile string) error

This method implements the saving function based on the method `GetFile`

Here is an example of saving a file:

```go
func (this *MainController) Post() {
	this.SaveToFile("the_file","/var/www/uploads/uploaded_file.txt")
}
```
## Data Bind

Data bind lets the user bind the request data to a variable, the request url as follow:

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
