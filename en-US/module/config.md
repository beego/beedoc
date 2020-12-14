---
name: Config Module
sort: 7
---

# Parsing Configuration Files

The config module is used for parsing configuration files, inspired by `database/sql`. It supports ini, json, xml and yaml files. You can install it by:

	go get github.com/beego/beego/v2/core/config

If you want to parse xml or yaml, you should first install:

	go get -u github.com/beego/beego/v2/core/config/xml

and then import:

	import _ "github.com/beego/beego/v2/core/config/xml"
	
# Remote configure middleware

Now we support `etcd` as the implementation.

# Usage

## Using package 

In v2.x, Beego create a `globalInstance`, so that users could use `config` module directly.

```go
val, err := config.String("mykey")
```

Beego use `ini` implementation and loads config from `config/app.conf`.

If the file not found or got some error, Beego outputs some warning log.

Or you can initialize the `globalInstance` by:

```go
_ import "github.com/beego/beego/v2/core/config/toml"
err := InitGlobalInstance("toml", "some config")
// ...
val, err := config.String("mykey")
// ...
```

## Create instance manually 

Initialize a parser object:

	iniconf, err := NewConfig("ini", "testini.conf")
	if err != nil {
		t.Fatal(err)
	}

Get data from parser:

	iniconf.String("appname")

### Parser methods

Here are the parser's methods:

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


Notice:
1. All `Default*` methods, default value will be returned if key not found or go some error;
2. `DIY` returns original value. When you want to use this method, you should be care of value's type. 
3. `GetSection` returns all configure items under the `section`. `section` has different meaning in different implementation.
4. `Unmarshaler` try to decode the configs to `obj`. And the parameter `prefix` is similar with `section`.
5. `Sub` is similar to `GetSection`, but `Sub` will wrap result as an `Configer` instance.
6. `Onchange` is used to listen change event. But most of file-based implementations don't support this method.
7. `SaveConfigFile` output configs to a file.
8. Some implementation support key like `a.b.c.d`, but not all implementations support it and not all of them use `.` as separator. 

### Configuration sections

The ini file supports configuration sections. You can get values inside a section by using `section::key`.

For example:

	[demo]
	key1 = "asta"
	key2 = "xie"

You can use `iniconf.String("demo::key2")` to get the value.

### How to Obtain Environment Variables

After Pull Request "Support get environment variables in config #1636" was merged into the code, beego supports using environment variables in the configuration file.

The format for this is `${ENVIRONMENTVARIABLE}` within the configuration file which is equivalent to `value = os.Getenv('ENVIRONMENTVARIABLE')`. Beego will only check for environment variables if the value begins with `${` and ends with `}`.

Additionally, a default value can be configured for the case that there is no environment variable set or the environment variable is empty. This is accomplished by using the format `${ENVVAR||defaultvalue}`, for example `${GOPATH||/home/asataxie/workspace/go}`. This `||` is used to split environment values and default values. See `/config/config_test.go` in the [beego repo](https://github.com/beego/beego) for more examples and edge cases about how these environment variables and default values are parsed.

For example:

	password = ${MyPWD}
	token = ${TOKEN||astaxie}
	user = ${MyUser||beego}

If the environment variable `$TOKEN` is set, its value will be used for the `token` configuration value and `beego.AppConfig.String("token")` would return its value. If `$TOKEN` is not set, the value would then be the string `astaxie`.

**Please note**: The environment variables are only read when the configuration file is parsed, not when configuration item is obtained by a function like `beego.AppConfig.String(string)`.
