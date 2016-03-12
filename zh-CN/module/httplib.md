---
name: httplib 模块
sort: 4
---

# 客户端请求
 
httplib 库主要用来模拟客户端发送 HTTP 请求，类似于 Curl 工具，支持 JQuery 类似的链式操作。使用起来相当的方便；通过如下方式进行安装：

	go get github.com/astaxie/beego/httplib

## 如何使用

首先导入包

	import (
		"github.com/astaxie/beego/httplib"
	)	

然后初始化请求方法，返回对象

	req := httplib.Get("http://beego.me/")

然后我们就可以获取数据了

	str, err := req.String()
	if err != nil {
		t.Fatal(err)
	}
	fmt.Println(str)
	
## 支持的方法对象

httplib 包里面支持如下的方法返回 request 对象：

- Get(url string)
- Post(url string)
- Put(url string)
- Delete(url string)
- Head(url string)

## 支持 debug 输出

可以根据上面五个方法返回的对象进行调试信息的输出：

	req.Debug(true)
	
这样就可以看到请求数据的详细输出
	
	httplib.Get("http://beego.me/").Debug(true).Response()
	
	//输出数据如下
	GET / HTTP/0.0
	Host: beego.me
	User-Agent: beegoServer

## 支持 HTTPS 请求

如果请求的网站是 HTTPS 的，那么我们就需要设置 client 的 TLS 信息，如下所示：

	req.SetTLSClientConfig(&tls.Config{InsecureSkipVerify: true})
	
关于如何设置这些信息请访问： http://gowalker.org/crypto/tls#Config			
	
## 支持超时设置

通过如下接口可以设置请求的超时时间和数据读取时间：

	req.SetTimeout(connectTimeout, readWriteTimeout)

以上方法都是针对 request 对象的，所以你第一步必须是返回 request 对象，然后链式操作，类似这样的代码：

	httplib.Get("http://beego.me/").SetTimeout(100 * time.Second, 30 * time.Second).Response()
	
## 设置请求参数

对于 Put 或者 Post 请求，需要发送参数，那么可以通过 Param 发送 k/v 数据，如下所示：

	req := httplib.Post("http://beego.me/")
	req.Param("username","astaxie")
	req.Param("password","123456")
	
## 发送大片的数据

有时候需要上传文件之类的模拟，那么如何发送这个文件数据呢？可以通过 Body 函数来操作，举例如下：
	
	req := httplib.Post("http://beego.me/")
	bt,err:=ioutil.ReadFile("hello.txt")
	if err!=nil{
		log.Fatal("read file err:",err)
	}
	req.Body(bt)
	
## 设置 header 信息

除了请求参数之外，我们有些时候需要模拟一些头信息，例如

	Accept-Encoding:gzip,deflate,sdch
	Host:beego.me
	User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36
	
可以通过 Header 函数来设置，如下所示：

	req := httplib.Post("http://beego.me/")
	req.Header("Accept-Encoding","gzip,deflate,sdch")
	req.Header("Host","beego.me")
	req.Header("User-Agent","Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36")
	
## httplib支持文件直接上传接口

PostFile 第一个参数是form 表单的字段名,第二个是需要发送的文件名或者文件路径

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
	
## 获取返回结果

上面这些都是在发送请求之前的设置，接下来我们开始发送请求，然后如何来获取数据呢？主要有如下几种方式：
- 返回 Response 对象，`req.Response()` 方法

	这个是 http.Response 对象，用户可以自己读取 body 的数据等。

- 返回 bytes,`req.Bytes()` 方法

	直接返回请求 URL 返回的内容

- 返回 string，`req.String()` 方法

	直接返回请求 URL 返回的内容
	
- 保存为文件，`req.ToFile(filename)` 方法

	返回结果保存到文件名为 filename 的文件中
	
- 解析为 JSON 结构，`req.ToJson(&result)` 方法

	返回结构直接解析为 JSON 格式，解析到 result 对象中
	
- 解析为 XML 结构，`req.ToXml(&result)` 方法

	返回结构直接解析为 XML 格式，解析到 result 对象中	
