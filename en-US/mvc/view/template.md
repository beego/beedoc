---
name: Template Functions
sort: 2
---

# Template Functions

Beego supports custom template functions but it must be set as below before `beego.Run()`:

	func hello(in string)(out string){
		out = in + "world"
		return
	}
	
	beego.AddFuncMap("hi",hello)

Then you can use these functions in template:

	{{.Content | hi}}

Here are Beego's buildin template functions:

* dateformat

  Format time, return string. {{dateformat .Time "2006-01-02T15:04:05Z07:00"}}

* date

  This is similar to date function in PHP. It can easily return time by string {{date .T "Y-m-d H:i:s"}}

* compare

  Compare two objects. If they are same return true otherwise return false. {{compare .A .B}}

* substr

  Return sub string. support UTF-8 string. {{substr .Str 0 30}}

* html2str

  Parse html to string by removing tags like script and css and return text. {{html2str .Htmlinfo}}

* str2html
  Parse string to HTML, no escaping. {{str2html .Strhtml}}

* htmlquote

  Escaping html. {{htmlquote .quote}}

* htmlunquote

  Unescaping to html. {{htmlunquote .unquote}}

* renderform

  Generate form from StructTag. {{&struct | renderform}}
