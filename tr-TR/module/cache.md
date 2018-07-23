---
name: Cache Module
sort: 2
---

# Cache Modülü

Beego'nun cache modülü veriyi cachelemek için kullanılır. `database/sql` paketinden ilham alınarak oluşturulan modül file, memcache, memory ve redis cache'i desteklemektedir. Yüklemek için aşağıdaki komutu çalıştırabilirsiniz : 

	go get github.com/astaxie/beego/cache

Eğer `memcache` veya `redis` kullanıyorsanız öncelikle aşağıdaki komutu çalıştırmalısınız : 

	go get -u github.com/astaxie/beego/cache/memcache

sonrasında proje dosyanızda kullanacağınız yere aşağıdaki paketi dahil etmelisiniz :

	import _ "github.com/astaxie/beego/cache/memcache"

## Basit Kullanım

İlk adım olarak proje dosyanızda kullanacağınız yere aşağıdaki paketi dahil etmelisiniz :

	import (
		"github.com/astaxie/beego/cache"
	)

Sonra global değişken olarak objeyi oluşturmalısınız :

	bm, err := cache.NewCache("memory", `{"interval":60}`)

Sonra `bm` olarak oluşturduğumuz caching değişkenini istediğimiz gibi düzenleyebiliriz :

	bm.Put("astaxie", 1, 10*time.Second)
	bm.Get("astaxie")
	bm.IsExist("astaxie")
	bm.Delete("astaxie")


## Sağlayıcı (Provider) Ayarları

Dört sağlayıcıyı aşağıdaki gibi ayarlayabiliriz : 

- memory

	`interval` GC zamanı anlamına gelir. Önbellek her 60 saniyede bir silinir :

		{"interval":60}

- file

		{"CachePath":"./cache","FileSuffix":".cache","DirectoryLevel":2,"EmbedExpiry":120}

- redis

	redis için [redigo](https://github.com/garyburd/redigo/tree/master/redis) kullanılmaktadır.

		{"key":"kolesiyonAdı","conn":":6039","dbNum":"0","password":"şifre"}

	* key: Redis collection adı
	* conn: Redis bağlantı bilgisi
	* dbNum: DB içerisinde bulunduğu index (sıfır tabanlı)
	* password: Şifreyle korunan Redis sunucusu için şifre


- memcache

	memcache için [vitess](http://code.google.com/p/vitess/go/memcache) paketi kullanılmaktadır.

		{"conn":"127.0.0.1:11211"}

## Kendi sağlayıcını kullanma

Cache modülü Cache interface'ini kullanmaktadır. Siz de bu interface'i kullanarak kendi cache sağlayınızı oluşturup kullanabilirsiniz.

	type Cache interface {
		Get(key string) interface{}
        GetMulti(keys []string) []interface{}
		Put(key string, val interface{}, timeout time.Duration) error
		Delete(key string) error
		Incr(key string) error
		Decr(key string) error
		IsExist(key string) bool
		ClearAll() error
		StartAndGC(config string) error
	}

Sağlayıcıyı kullanmak için:

	func init() {
		cache.Register("kendicacheim", NewOwnCache())
	}
