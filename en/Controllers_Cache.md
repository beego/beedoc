# Cache

Beego has a built-in cache module, it's like memcache, which caches data in memory. Here is an example of using cache module in Beego:

	var (
		urllist *beego.BeeCache
	)
	
	func init() {
		urllist = beego.NewBeeCache()
		urllist.Every = 0 // Not expired
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

To use cache, you need to initialize a `beego.NewBeeCache` object and set expired time, and enable expired check. Then you can use following methods to achieve other operations:

- Get(name string) interface{}
- Put(name string, value interface{}, expired int) error
- Delete(name string) (ok bool, err error)
- IsExist(name string) bool
