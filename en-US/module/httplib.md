---
name: Httplib Module
sort: 4
---

# Client Request

Similar to Curl, httplib is used to simulate http requests sent by client. Similar to JQuery, it supports chain calls. It's easy to use. Can be installed by:

	go get github.com/astaxie/beego/httplib

## Basic Usage

Import package:

	import (
		"github.com/astaxie/beego/httplib"
	)	

Initialize request method and return response:

	req:=httplib.Get("http://beego.me/")

Then we can use the data in response:

	str, err := req.String()
	if err != nil {
		t.Fatal(err)
	}
	fmt.Println(str)
	
## Method Functions

httplib supports these methods:

- Get(url string)
- Post(url string)
- Put(url string)
- Delete(url string)
- Head(url string)

## Debug Output

Enable debug information output:

	req.Debug(true)
	
Then it will output debug infomation:
	
	Get("http://beego.me/").Debug(true).Response()
	
	// Output
	GET / HTTP/0.0
	Host: beego.me
	User-Agent: beegoServer

## HTTPS Request

If the requested scheme is https, we need to set TLS of client:

	httplib.SetTLSClientConfig(&tls.Config{InsecureSkipVerify: true})
	
[Learn more about TLS settings](http://gowalker.org/crypto/tls#Config)
	
## Set Timeout

Can set request timeout and data reading timeout by:

	SetTimeout(connectTimeout, readWriteTimeout)

It is a function of request object. So it can be done like this:

	Get("http://beego.me/").SetTimeout(100 * time.Second, 30 * time.Second).Response()
	
## Set Request Params

For Put or Post request, we may need to send params. Params can be set like:

	req:=httplib.Post("http://beego.me/")
	req.Param("username","astaxie")
	req.Param("password","123456")
	
## Send big data

To simulate file uploading or to send big data, can use `Body` function:
	
	req:=httplib.Post("http://beego.me/")
	bt,err:=ioutil.ReadFile("hello.txt")
	if err!=nil{
		log.Fatal("read file err:",err)
	}
	req.Body(bt)
	
## Set header

To simulate header values, E.g.:

	Accept-Encoding:gzip,deflate,sdch
	Host:beego.me
	User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36
	
Can use `Header` function:

	req:=httplib.Post("http://beego.me/")
	req.Header("Accept-Encoding","gzip,deflate,sdch")
	req.Header("Host","beego.me")
	req.Header("User-Agent","Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36")
	
## upload file

PostFile function the first params is the name of form, the second param is the filename or filepath you want to send. 

```
b:=httplib.Post("http://beego.me/")
b.Param("username","astaxie")
b.Param("password","123456")
b.PostFile("uploadfile1", "httplib.pdf")
b.PostFile("uploadfile2", "httplib.txt")
str, err := b.String()
if err != nil {
    t.Fatal(err)
}	
```

		
## Get Response 

Above settings are before sending request, how can we get response after request? Here are the ways:

- return Response object by `req.Response()`

  This is a http.Response object. You can get data from it.

- return bytes by `req.Bytes()`

  Return raw response.

- return string by `req.String()`

  Return raw response.
	
- save file by `req.ToFile(filename)`

  Save response into file.
	
- parse to JSON by `req.ToJson(result)`

  Parse response to JSON into result object
	
- parse to xml by `req.ToXML(result)`

  Parse response to XML into result object
