---
name: Response formats
sort: 8
---

# JSON, XML and JSONP

Beego is also designed for the creation of API applications. When we build an API application, we often need to respond with JSON or XML. Beego provides a simple approach:

- Respond with JSON data:

	```go
	type mystruct struct {
	  FieldOne string `json:"field_one"`
	}
	
	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["json"] = &mystruct
		this.ServeJSON()
	}
	```
  ServeJson will set `content-type` to `application/json` and JSONify the data.

- Respond with XML data:
	
	```go
	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["xml"]=&mystruct
		this.ServeXml()
	}
	```
  ServeXml will set `content-type` to `application/xml` and convert the data into XML.

- Respond with jsonp

	```go
	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["jsonp"] = &mystruct
		this.ServeJSONP()
	}
	```
  ServeJsonp will set `content-type` to `application/javascript` , JSONify the data and respond to jsonp based on the request parameter `callback`.

In version 1.6 names of methods were changed, it is ServeJSON(), ServeXML(), ServeJSONP() from now on.
