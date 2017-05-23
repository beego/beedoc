---
name: Cache Module
sort: 2
---

# Cache Module

Beego's cache module is used for caching data, inspired by `database/sql`. It supports four cache providers: file, memcache, memory and redis. You can install it by:

	go get github.com/astaxie/beego/cache

If you use the `memcache` or `redis` provider, you should first install:

	go get -u github.com/astaxie/beego/cache/memcache

and then import:

	import _ "github.com/astaxie/beego/cache/memcache"

## Basic Usage

First step is importing the package:

	import (
		"github.com/astaxie/beego/cache"
	)

Then initialize a global variable object:

	bm, err := cache.NewCache("memory", `{"interval":60}`)

Then we can use `bm` to modify the cache:

	bm.Put("astaxie", 1, 10)
	bm.Get("astaxie")
	bm.IsExist("astaxie")
	bm.Delete("astaxie")

## Provider Settings

Here is how to configure the four providers:

- memory

	`interval` stands for GC time, which means the cache will be cleared every 60s:

		{"interval":60}

- file

		{"CachePath":"./cache","FileSuffix":".cache","DirectoryLevel":2,"EmbedExpiry":120}

- redis

	redis is using [redigo](http://github.com/garyburd/redigo/redis)

		{"conn":":6039"}

- memcache

	memcache is using [vitess](http://code.google.com/p/vitess/go/memcache)

		{"conn":"127.0.0.1:11211"}

## Creating your own provider

The cache module uses the Cache interface, so you can create your own cache provider by implementing this interface and registering it.

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

Register your provider:

	func init() {
		cache.Register("myowncache", NewOwnCache())
	}
