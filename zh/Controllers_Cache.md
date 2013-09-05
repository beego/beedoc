# 数据缓存（Cache）

beego 内置了一个 cache 模块，实现了类似 memcache 的功能，缓存数据在内存中，主要的使用方法如下：

```go
var (
	urllist *beego.BeeCache
)

func init() {
	urllist = beego.NewBeeCache()
	urllist.Every = 0 //不过期
	urllist.Start()
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

上面这个例子演示了如何使用 beego 的 Cache 模块，主要是通过 `beego.NewBeeCache` 初始化一个对象，然后设置过期时间，开启过期检测，在业务逻辑中就可以通过如下的接口进行增删改的操作：

- Get(name string) interface{}
- Put(name string, value interface{}, expired int) error
- Delete(name string) (ok bool, err error)
- IsExist(name string) bool