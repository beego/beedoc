---
name: Config Module
sort: 7
---

# Parsing Configuration Files

This module is used for parsing file, inspired by `database/sql`. It
supports ini, json, xml, yaml files. You can install it by:

	go get github.com/astaxie/beego/config
	
>>>if you want to parse xml or yaml. you should install first

	go get -u github.com/astaxie/beego/config/xml
	
>>>then import it

    import _ "github.com/astaxie/beego/config/xml"	
	
## Basic Usage

Initialize a parser object:

	iniconf, err := NewConfig("ini", "testini.conf")
	if err != nil {
		t.Fatal(err)
	}
	
Get data by parser:
	
	iniconf.String("appname")
	
Here is the parser's methods:

* Set(key, val string) error
* String(key string) string
* Int(key string) (int, error)
* Int64(key string) (int64, error)
* Bool(key string) (bool, error)
* Float(key string) (float64, error)
* DIY(key string) (interface{}, error)
	
The ini config file supports section. You can get value by `section::key`

For example:

  [demo]
  key1="asta"
  key2 = "xie"

You can use `iniconf.String("demo::key2")` to get the value
