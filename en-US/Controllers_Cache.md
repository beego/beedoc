# Cache

Beego uses modular design so that cache becomes an independent module, and you can use following command to install it:

	go get github.com/astaxie/beego/cache

For now, it implemented 3 design of engines: memory, memcahe, redis.

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
Above example shows how to use cache module in beego, where you need to initialize an object through `cache.NewCache` before you are allowed to use following operations:

* Get(key string) interface{}
* Put(key string, val interface{}, timeout int64) error
* Delete(key string) error
* Incr(key string) error
* Decr(key string) error
* IsExist(key string) bool
* ClearAll() error

The configuration of memcahe and redis are similar, the only difference is the initialize information that pass to the function `cache.NewCache`; for example:

	{"conn":"127.0.0.1:11211"}