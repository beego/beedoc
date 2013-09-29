# 配置文件解析

beego目前采用了模块化设计，config独立出来了一个模块，你可以通过如下方式进行安装：

	go get github.com/astaxie/beego/config
	
目前实现了四个文件格式的解析：ini、json、xml、yaml

具体的使用例子如下所示：	


```go

func Post() {
	iniconf, err := config.NewConfig("ini", "testini.conf")
	if err != nil {
		t.Fatal(err)
	}
	if iniconf.String("appname") != "beeapi" {
		t.Fatal("appname not equal to beeapi")
	}
	if port, err := iniconf.Int("httpport"); err != nil || port != 8080 {
		t.Error(port)
		t.Fatal(err)
	}
	if port, err := iniconf.Int64("mysqlport"); err != nil || port != 3600 {
		t.Error(port)
		t.Fatal(err)
	}
	if pi, err := iniconf.Float("PI"); err != nil || pi != 3.1415976 {
		t.Error(pi)
		t.Fatal(err)
	}
	if iniconf.String("runmode") != "dev" {
		t.Fatal("runmode not equal to dev")
	}
	if v, err := iniconf.Bool("autorender"); err != nil || v != false {
		t.Error(v)
		t.Fatal(err)
	}
	if v, err := iniconf.Bool("copyrequestbody"); err != nil || v != true {
		t.Error(v)
		t.Fatal(err)
	}
	if err = iniconf.Set("name", "astaxie"); err != nil {
		t.Fatal(err)
	}
	if iniconf.String("name") != "astaxie" {
		t.Fatal("get name error")
	}
}
```

上面这个例子演示了如何使用 beego 的 config 模块，主要是通过 `config.NewConfig` 初始化一个对象，在业务逻辑中就可以通过如下的接口进行数据的读取：

* Set(key, val string) error
* String(key string) string
* Int(key string) (int, error)
* Int64(key string) (int64, error)
* Bool(key string) (bool, error)
* Float(key string) (float64, error)
* DIY(key string) (interface{}, error)

config模块中最重要的是如下这个函数:

	config.NewConfig(adapterName, fileaname string)
	
- 第一个参数表示解析的文件格式，目前支持四种：ini、json、xml、yaml
- 第二个参数表示文件名