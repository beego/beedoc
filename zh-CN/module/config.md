---
name: config 模块
sort: 7
---

# 配置文件解析

这是一个用来解析文件的库，它的设计思路来自于 `database/sql`，目前支持解析的文件格式有 ini、json、xml、yaml，可以通过如下方式进行安装：

	go get github.com/beego/beego/v2/core/config

>>>如果你使用xml 或者 yaml 驱动就需要手工安装引入包

	go get -u github.com/beego/beego/v2/core/config/xml

>>>而且需要在使用的地方引入包

    import _ "github.com/beego/beego/v2/core/config/xml"

# 远程配置

目前我们提供了对远程配置中心`etcd`的支持。使用它就如同使用一般的基于文件的配置那样。

# 如何使用

## 直接使用包

为了简化代码，我们在`config`初始化的时候，默认创建了一个名为`globalInstance`的实例。

该实例在没有显式初始化的情况下，默认是采用`ini`作为实现，它将会加载文件`conf/app.conf`。

在加载失败的时候，会输出一个警告信息。

于是我们可以直接使用`config`包。

```go
val, err := config.String("mykey")
```

当然，用户也可以主动初始化。例如，当我们想用`toml`作为默认实现的时候，我们可以：

```go
_ import "github.com/beego/beego/v2/core/config/toml"
err := InitGlobalInstance("toml", "some config")
// ...
val, err := config.String("mykey")
// ...
```

永远别忘了引入具体实现的包。

## 手动创建实例

首先初始化一个解析器对象

```go
iniconf, err := NewConfig("ini", "testini.conf")
if err != nil {
	t.Fatal(err)
}
```

然后通过对象获取数据

	iniconf.String("appname")

解析器对象支持的函数有如下：

```go
// Configer defines how to get and set value from configuration raw data.
type Configer interface {
	// support section::key type in given key when using ini type.
	Set(key, val string) error

	// support section::key type in key string when using ini and json type; Int,Int64,Bool,Float,DIY are same.
	String(key string) (string, error)
	// get string slice
	Strings(key string) ([]string, error)
	Int(key string) (int, error)
	Int64(key string) (int64, error)
	Bool(key string) (bool, error)
	Float(key string) (float64, error)
	// support section::key type in key string when using ini and json type; Int,Int64,Bool,Float,DIY are same.
	DefaultString(key string, defaultVal string) string
	// get string slice
	DefaultStrings(key string, defaultVal []string) []string
	DefaultInt(key string, defaultVal int) int
	DefaultInt64(key string, defaultVal int64) int64
	DefaultBool(key string, defaultVal bool) bool
	DefaultFloat(key string, defaultVal float64) float64

	// DIY return the original value
	DIY(key string) (interface{}, error)

	GetSection(section string) (map[string]string, error)

	Unmarshaler(prefix string, obj interface{}, opt ...DecodeOption) error
	Sub(key string) (Configer, error)
	OnChange(key string, fn func(value string))
	SaveConfigFile(filename string) error
}
```

这里有一些使用的注意事项：

1. 所有的`Default*`方法，在`key`不存在，或者查找的过程中，出现`error`，都会返回默认值；
2. `DIY`直接返回对应的值，而没有做任何类型的转换。当你使用这个方法的时候，你应该自己确认值的类型。只有在极少数的情况下你才应该考虑使用这个方法；
3. `GetSection`会返回`section`所对应的部分配置。`section`如何被解释，取决于具体的实现；
4. `Unmarshaler`会尝试用当且配置的值来初始化`obj`。需要注意的是，`prefix`的概念类似于`section`；
5. `Sub`类似与`GetSection`，都是尝试返回配置的一部分。所不同的是，`GetSection`将结果组织成`map`，而`Sub`将结果组织成`Config`实例；
6. `OnChange`主要用于监听配置的变化。对于大部分依赖于文件系统的实现来说，都不支持。具体而言，我们设计这个主要是为了考虑支持远程配置；
7. `SaveConfigFile`尝试将配置导出成为一个文件；
8. 某些实现支持分段式的`key`。比如说`a.b.c`这种，但是，并不是所有的实现都支持，也不是所有的实现都采用`.`作为分隔符。这是一个历史遗留问题，为了保留兼容性，我们无法在这方面保持一致。

这里额外提及一下`ini`文件，因为这是早期v1.x里面默认的配置实现：

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
