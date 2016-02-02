---
name: 多种格式数据输出
sort: 8
---

# JSON、XML、JSONP

beego 当初设计的时候就考虑了 API 功能的设计，而我们在设计 API 的时候经常是输出 JSON 或者 XML 数据，那么 beego 提供了这样的方式直接输出：

注意struct属性应该 为 exported Identifier
首字母应该大写
- JSON 数据直接输出：

	```go
	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["json"] = &mystruct
		this.ServeJSON()
	}
	```
	调用 ServeJSON 之后，会设置 `content-type` 为 `application/json`，然后同时把数据进行 JSON 序列化输出。

- XML 数据直接输出：
	
	```go
	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["xml"]=&mystruct
		this.ServeXML()
	}
	```
	调用 ServeXML 之后，会设置 `content-type` 为 `application/xml`，同时数据会进行 XML 序列化输出。

- jsonp 调用

	```go
	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["jsonp"] = &mystruct
		this.ServeJSONP()
	}
	```
	调用 ServeJSONP 之后，会设置 `content-type` 为 `application/javascript`，然后同时把数据进行 JSON 序列化，然后根据请求的 callback 参数设置 jsonp 输出。
	
	
开发模式下序列化后输出的是格式化易阅读的 JSON 或 XML 字符串；在生产模式下序列化后输出的是压缩的字符串。
