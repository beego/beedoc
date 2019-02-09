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

### Parser methods

Here are the parser's methods:

* **Setting values:**
	* `Set(key, val string) error`
	* `SaveConfigFile(filename string) error`
* **Getting values:**
	* `String(key string) string`
	* `Strings(key string) []string`
	* `Int(key string) (int, error)`
	* `Int64(key string) (int64, error)`
	* `Bool(key string) (bool, error)`
	* `Float(key string) (float64, error)`
	* `DIY(key string) (interface{}, error)`
	* `GetSection(section string) (map[string]string, error)`
* **Getting values or a default value:**
	* `DefaultString(key string, defaultval string) string`
	* `DefaultStrings(key string, defaultval []string) []string`
	* `DefaultInt(key string, defaultval int) int`
	* `DefaultInt64(key string, defaultval int64) int64`
	* `DefaultBool(key string, defaultval bool) bool`
	* `DefaultFloat(key string, defaultval float64) float64`

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

Additionally, a default value can be configured for the case that there is no environment variable set or the environment variable is empty. This is accomplished by using the format `${ENVVAR||defaultvalue}`, for example `${GOPATH||/home/asataxie/workspace/go}`. This `||` is used to split environment values and default values. See `/config/config_test.go` in the [beego repo](https://github.com/astaxie/beego) for more examples and edge cases about how these environment variables and default values are parsed.

For example:

	password = ${MyPWD}
	token = ${TOKEN||astaxie}
	user = ${MyUser||beego}

If the environment variable `$TOKEN` is set, its value will be used for the `token` configuration value and `beego.AppConfig.String("token")` would return its value. If `$TOKEN` is not set, the value would then be astaxie.

**Please note**: The environment variables are only read when the configuration file is parsed, not when configuration item is obtained by a function.
