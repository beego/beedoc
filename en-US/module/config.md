---
name: Config Module
sort: 7
---

# Parsing Configuration Files

The config module is used for parsing configuration files, inspired by `database/sql`. It supports ini, json, xml and yaml files. You can install it by:

	go get github.com/astaxie/beego/config

If you want to parse xml or yaml, you should first install:

	go get -u github.com/astaxie/beego/config/xml

and then import:

	import _ "github.com/astaxie/beego/config/xml"

## Basic Usage

Initialize a parser object:

	iniconf, err := NewConfig("ini", "testini.conf")
	if err != nil {
		t.Fatal(err)
	}

Get data from parser:

	iniconf.String("appname")

Here are the parser's methods:

* `Set(key, val string) error`
* `String(key string) string`
* `Int(key string) (int, error)`
* `Int64(key string) (int64, error)`
* `Bool(key string) (bool, error)`
* `Float(key string) (float64, error)`
* `DIY(key string) (interface{}, error)`

The ini file supports configuration sections. You can get values inside a section by using `section::key`.

For example:

	[demo]
	key1="asta"
	key2 = "xie"

You can use `iniconf.String("demo::key2")` to get the value.
