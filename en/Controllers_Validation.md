# Validation

validation is a form validation for a data validation and error collecting using Go.

### Installation and tests

Install:

	go get github.com/astaxie/beego/validation

Test:

	go test github.com/astaxie/beego/validation

### Example

Direct Use:

	import (
		"github.com/astaxie/beego/validation"
		"log"
	)

	type User struct {
		Name string
		Age int
	}

	func main() {
		u := User{"man", 40}
		valid := validation.Validation{}
		valid.Required(u.Name, "name")
		valid.MaxSize(u.Name, 15, "nameMax")
		valid.Range(u.Age, 0, 140, "age")
		if valid.HasErrors {
			// validation does not pass
			// print invalid message
			for _, err := range valid.Errors {
				log.Println(err.Key, err.Message)
			}
		}
		// or use like this
		if v := valid.Max(u.Age, 140); !v.Ok {
			log.Println(v.Error.Key, v.Error.Message)
		}
	}

Struct Tag Use:

	import (
		"github.com/astaxie/beego/validation"
	)

	// validation function follow with "valid" tag
	// functions divide with ";"
	// parameters in parentheses "()" and divide with ","
	// Match function's pattern string must in "//"
	type user struct {
		Id   int
		Name string `valid:"Required;Match(/^(test)?\\w*@;com$/)"`
		Age  int    `valid:"Required;Range(1, 140)"`
	}

	func main() {
		valid := Validation{}
		u := user{Name: "test", Age: 40}
		b, err := valid.Valid(u)
		if err != nil {
			// handle error
		}
		if !b {
			// validation does not pass
			// blabla...
		}
	}

Struct Tag Functions:

* `Required` Non-empty, which means value cannot be its type's zero-value.
* `Min(min int)` Minimum value with type that has to be `int`.
* `Max(max int)` Maximum value with type that has to be `int`.
* `Range(min, max int)` Range of value with type that has to be `int`.
* `MinSize(min int)` Minimum length of value with type that has to be `string` or `slice`.
* `MaxSize(max int)` Maximum length of value with type that has to be `string` or `slice`.
* `Length(length int)` Certain length of value with type that has to be `string` or `slice`.
* `Alpha` alpha characters with type that has to be `string`.
* `Numeric` number with type that has to be `string`.
* `AlphaNumeric` alpha characters or number with type that has to be `string`.
* `Match(pattern string)` Regexp match with type that has to be `string`, other types will be converted by `fmt.Sprintf("%v", obj).Match` before matching.
* `AlphaDash` alpha characters or number or `-_` with type that has to be `string`.
* `Email` email format with type that has to be `string`.
* `IP`  IP format with type that has to be `string`, for now only support IPv4.
* `Base64` base64 encode with type that has to be `string`.
* `Mobile` phone number with type that has to be `string`.
* `Tel` telephone number with type that has to be `string`.
* `Phone` phone or telephone number with type that has to be `string`.
* `ZipCode` zip code with type that has to be `string`.

### API documnetation

Please visit [Go Walker](http://gowalker.org/github.com/astaxie/beego/validation).