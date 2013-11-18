# 数据缓存（Cache）

beego 目前采用了模块化设计，cache 独立出来了一个模块，你可以通过如下方式进行安装：

	go get github.com/astaxie/beego/cache
	
目前实现了三个引擎的设计：memory、memcahe、redis	


```go
var (
	urllist *cache.Cache
)

func init() {
	urllist, err := cache.NewCache("memory", `{"interval":60}`)
	if err!=nil{
		//抛出异常
	}
}

func (this *ShortController) Post() {
	var result ShortResult
	longurl := this.Input().Get("longurl")
	beego.Info(longurl)
	result.UrlLong = longurl
	urlmd5 := models.GetMD5(longurl)
	beego.Info(urlmd5)
	if urllist.IsExist(urlmd5) {
		result.UrlShort = urllist.Get(urlmd5).(string)
	} else {
		result.UrlShort = models.Generate()
		err := urllist.Put(urlmd5, result.UrlShort, 0)
		if err != nil {
			beego.Info(err)
		}
		err = urllist.Put(result.UrlShort, longurl, 0)
		if err != nil {
			beego.Info(err)
		}
	}
	this.Data["json"] = result
	this.ServeJson()
}
```

上面这个例子演示了如何使用 beego 的 Cache 模块，主要是通过 `cache.NewCache` 初始化一个对象，在业务逻辑中就可以通过如下的接口进行增删改的操作：

* Get(key string) interface{}
* Put(key string, val interface{}, timeout int64) error
* Delete(key string) error
* Incr(key string) error
* Decr(key string) error
* IsExist(key string) bool
* ClearAll() error

memcahe和redis的配置也是类似，只是后面的初始化配置信息不一样，这两者的配置信息是如下包含了链接信息：

	{"conn":"127.0.0.1:11211"}