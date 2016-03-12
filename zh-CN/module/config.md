---
name: config 模块
sort: 7
---

# 配置文件解析

这是一个用来解析文件的库，它的设计思路来自于 `database/sql`，目前支持解析的文件格式有 ini、json、xml、yaml，可以通过如下方式进行安装：

	go get github.com/astaxie/beego/config
	
>>>如果你使用xml 或者 yaml 驱动就需要手工安装引入包

	go get -u github.com/astaxie/beego/config/xml
	
>>>而且需要在使用的地方引入包

    import _ "github.com/astaxie/beego/config/xml"				
	
## 如何使用

首先初始化一个解析器对象

	iniconf, err := NewConfig("ini", "testini.conf")
	if err != nil {
		t.Fatal(err)
	}
	
然后通过对象获取数据
	
	iniconf.String("appname")
	
解析器对象支持的函数有如下：

* Set(key, val string) error
* String(key string) string
* Int(key string) (int, error)
* Int64(key string) (int64, error)
* Bool(key string) (bool, error)
* Float(key string) (float64, error)
* DIY(key string) (interface{}, error)
	
>>>ini 配置文件支持 section 操作，key通过 `section::key` 的方式获取

>>> 例如下面这样的配置文件

>>>		[demo]
>>>		key1 = "asta"
>>>		key2 = "xie"

>>> 那么可以通过 `iniconf.String("demo::key2")` 获取值
