---
name: Response formats
sort: 8
---

# JSON, XML and JSONP

Beego is also designed for API application. When we build API application, we often need to respond with JSON or XML. Beego provides a simple approach:

- Respond with JSON data:

	```go
	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["json"] = &mystruct
		this.ServeJson()
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
  ServeJson will set `content-type` to `application/xml` and convert the data into XML.

- Respond with jsonp

	```go
	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["jsonp"] = &mystruct
		this.ServeJsonp()
	}
	```
  ServeJsonp will set `content-type` to `application/javascript` and JSONify the data and respond to jsonp based on the request parameter `callback`.
