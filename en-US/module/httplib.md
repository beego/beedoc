---
name: Httplib Module
sort: 4
---

# Client Request

Similar to Curl, httplib is used to simulate http requests sent by clients. Similar to jQuery, it supports method chaining. It's easy to use and it can be installed by:

	go get github.com/beego/beego/v2/client/httplib

## Basic Usage

Import package:

	import (
		"github.com/beego/beego/v2/client/httplib"
	)	

Initialize request method and url:

	req := httplib.Get("http://beego.me/")

Send the request and retrieve the data in the response:

	str, err := req.String()
	if err != nil {
		t.Fatal(err)
	}
	fmt.Println(str)
	
## Method Functions

httplib supports these methods:

- `Get(url string)`
- `Post(url string)`
- `Put(url string)`
- `Delete(url string)`
- `Head(url string)`

## Debug Output

Enable debug information output:

	req.Debug(true)

Then it will output debug information:

	httplib.Get("http://beego.me/").Debug(true).Response()

	// Output
	GET / HTTP/0.0
	Host: beego.me
	User-Agent: beegoServer

## HTTPS Request

If the requested scheme is https, we need to set the TLS of client:

	req.SetTLSClientConfig(&tls.Config{InsecureSkipVerify: true})

[Learn more about TLS settings](http://gowalker.org/crypto/tls#Config)

## Set Timeout

Can set request timeout and data reading timeout by:

	req.SetTimeout(connectTimeout, readWriteTimeout)

It is a function of request object. So it can be done like this:

	httplib.Get("http://beego.me/").SetTimeout(100 * time.Second, 30 * time.Second).Response()
	
## Set Request Params

For Put or Post requests, we may need to send parameters. Parameters can be set in the following manner:

	req := httplib.Post("http://beego.me/")
	req.Param("username","astaxie")
	req.Param("password","123456")

## Send big data

To simulate file uploading or to send big data, one can use the `Body` function:

	req := httplib.Post("http://beego.me/")
	bt,err:=ioutil.ReadFile("hello.txt")
	if err!=nil{
		log.Fatal("read file err:",err)
	}
	req.Body(bt)

## Set header

To simulate header values, e.g.:

	Accept-Encoding:gzip,deflate,sdch
	Host:beego.me
	User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36

Can use `Header` function:

	req := httplib.Post("http://beego.me/")
	req.Header("Accept-Encoding","gzip,deflate,sdch")
	req.Header("Host","beego.me")
	req.Header("User-Agent","Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36")

## Upload file

PostFile function requires the first parameter to be the name of form and the second parameter is the filename or filepath you want to send. 

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

The settings above are before sending request, how can we get response after request? Here are the ways:

|Method                          |Type                     |Description                                                |
|--------------------------------|-------------------------|-----------------------------------------------------------|
|`req.Response()`                |`(*http.Response, error)`|This is a `http.Response` object. You can get data from it.|
|`req.Bytes()`                   |`([]byte, error)`        |Return raw response body.                                  |
|`req.String()`                  |`(string, error)`        |Return raw response body.                                  |
|`req.ToFile(filename string)`   |`error`                  |Save response body into a file.                            |
|`req.ToJSON(result interface{})`|`error`                  |Parse JSON response into the result object.                |
|`req.ToXml(result interface{})` |`error`                  |Parse XML response into the result object.                 |


# Filter

In order to support some AOP feature, e.g. logs, tracing, we designed `filter-chain` for httplib.

There are two key interfaces:

```go
type FilterChain func(next Filter) Filter

type Filter func(ctx context.Context, req *BeegoHTTPRequest) (*http.Response, error)
```

This is a typical usage of `Filter-Chain` pattern. So you must invoke `next(...)` when you want to implement your own logic.

Here is an example：
```go
func myFilter(next httplib.Filter) httplib.Filter {
	return func(ctx context.Context, req *httplib.BeegoHTTPRequest) (*http.Response, error) {
		r := req.GetRequest()
		logs.Info("hello, here is the filter: ", r.URL)
		// Never forget invoke this. Or the request will not be sent
		return next(ctx, req)
	}
}
```

And we could register this filter as global filter:
```go
	httplib.SetDefaultSetting(httplib.BeegoHTTPSettings{

		FilterChains: []httplib.FilterChain{
			myFilter,
		},

		UserAgent:        "beegoServer",
		ConnectTimeout:   60 * time.Second,
		ReadWriteTimeout: 60 * time.Second,
		Gzip:             true,
		DumpBody:         true,
	})
```

Sometimes you only want to use the filter for specific requests:

```go
req.AddFilters(myFilter)
```

We provide some filters.

## Prometheus Filter

It's used to support `Prometheus` framework to collect metric data.

```go
	builder := prometheus.FilterChainBuilder{
		AppName: "My-test",
		ServerName: "User-server-1",
		RunMode: "dev",
	}
	req := httplib.Get("http://beego.me/")
	// only work for this request, or using SetDefaultSetting to support all requests
	req.AddFilters(builder.FilterChain)

	resp, err := req.Response()
	if err != nil {
		logs.Error("could not get response: ", err)
	} else {
		logs.Info(resp)
	}
```

If you don't use Beego's admin service, you must expose `prometheus` port manually.


## Opentracing Filter

```go
	builder := opentracing.FilterChainBuilder{}
	req := httplib.Get("http://beego.me/")
	// only work for this request, or using SetDefaultSetting to support all requests
	req.AddFilters(builder.FilterChain)

	resp, err := req.Response()
	if err != nil {
		logs.Error("could not get response: ", err)
	} else {
		logs.Info(resp)
	}
```

Don't forget to register `Opentracing` real implementation.
