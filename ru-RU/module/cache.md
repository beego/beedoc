---
name: Модуль кеширования
sort: 2
---

# Модуль кеширования

Модуль кеширования BeeGo используется для кеширования данных, вдохновлен модулем `database/sql`. Поддерживает 4 кеш провайдера: file, memcache, memory и redis. Вы можете установить модуль следующей командой:

	go get github.com/astaxie/beego/cache

Если вы используете `memcache` или `redis` провайдер, вы должны для начала установить их:

	go get -u github.com/astaxie/beego/cache/memcache

а затем импортировать:

	import _ "github.com/astaxie/beego/cache/memcache"

## Базовое использование

Первым шагом импортируйте пакет:

	import (
		"github.com/astaxie/beego/cache"
	)

Затем инициализируйте глобальную переменную типа объект:

	bm, err := cache.NewCache("memory", `{"interval":60}`)

Затем мы можем использовать `bm` чтобы модифицировать кеш:

	bm.Put("astaxie", 1, 10)
	bm.Get("astaxie")
	bm.IsExist("astaxie")
	bm.Delete("astaxie")

## Настройка провайдера

Ниже how to которое показывает как настроить 4 провайдера:

- memory

	`interval` stands for GC time, which means the cache will be cleared every 60s:

		{"interval":60}

- file

		{"CachePath":"./cache","FileSuffix":".cache","DirectoryLevel":2,"EmbedExpiry":120}

- redis

	redis в действии [redigo](http://github.com/garyburd/redigo/redis)

		{"conn":":6039"}

- memcache

	memcache в действии [vitess](http://code.google.com/p/vitess/go/memcache)

		{"conn":"127.0.0.1:11211"}

## Создание собственного провайдера

Кеш модуль использует интерфейс Cache, таким образом вы можете создать ваш собественный кеш провайдер, реализую этот интерфейс и зарегистрировав его.

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

Зарегистрируйте ваш провайдер:

	func init() {
		cache.Register("myowncache", NewOwnCache())
	}
