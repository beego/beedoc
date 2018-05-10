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

>>> 那么可以通过 `iniconf.String("demo::key2")` 获取值.
>>>
>>> ## 如何获取环境变量
>>> config 模块支持环境变量配置，对应配置项 Key 格式为 `${环境变量名}` ，则 Value = os.Getenv('环境变量名')。
>>> 同时可配置默认值，当环境变量无此配置或者环境变量值为空时，则优先使用默认值。包含默认值的 Key 格式为 `${GOPATH||/home/workspace/go/}` ，使用`||`分割环境变量和默认值。
>>>
>>> **注意**  获取环境变量值仅仅是在配置文件解析时处理，而不会在调用函数获取配置项时实时处理。
