---
name: Cache Module
sort: 2
---

# Cache Module

Beego's cache module is used for cache data, inspired by `databae/sql`.  It supports four cache providers: file, memcache, memory and redis. You can install it by:

	go get github.com/astaxie/beego/cache
	
>>>if you use memcache or redis provider. You should install first

	go get -u github.com/astaxie/beego/cache/memcache
	
>>>then import it.

    import _ "github.com/astaxie/beego/cache/memcache"	
	
## Basic Usage

First step is importing the package:

	import (
		"github.com/astaxie/beego/cache"
	)

Then initialize a global variable object:

	bm, err := cache.NewCache("memory", `{"interval":60}`)

Then we can use `bm` to modify cache:

	bm.Put("astaxie", 1, 10)
	bm.Get("astaxie")
	bm.IsExist("astaxie")
	bm.Delete("astaxie")

## Provider Settings

Here is how to set four providers:

- memory

  `interval` stands for GC time, which means every 60s will clear the cache:
	
  {"interval":60}													

- file

	
		{"CachePath":"./cache","FileSuffix":".cache","DirectoryLevel":2,"EmbedExpiry":120}
		
- redis

	redis is using [redigo](http://github.com/garyburd/redigo/redis)
	
		{"conn":":6039"}
		
- memcache

  memcache is using [vitess](http://code.google.com/p/vitess/go/memcache)
	
		{"conn":"127.0.0.1:11211"}	
		
## Create your own provider

cache module implemented the Cache interface, so you can create your own cache provider by implementing this interface and registering it.

	type Cache interface {
		Get(key string) interface{}
		Put(key string, val interface{}, timeout int64) error
		Delete(key string) error
		Incr(key string) error
		Decr(key string) error
		IsExist(key string) bool
		ClearAll() error
		StartAndGC(config string) error
	}		

Register your provider

	func init() {
		Register("myowncache", NewOwnCache())
	}
		
