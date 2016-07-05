---
name: Form validation
sort: 9
---

# Form validation

The Form validation module is used for data validation and error collection.

## Installing and testing

Installing:

	go get github.com/astaxie/beego/validation

Testing:

	go test github.com/astaxie/beego/validation

## Localization

In order to localize validation error messages, one might use `SetDefaultMessage` function of the `validation` package.

Note that format markers (`%d`, `%s`) must be preserved in translated text to provide resulting messages with validation context values.

Default template messages are present in `validation.MessageTmpls` variable.

Simple message localization for Russian language:

```go
import "github.com/astaxie/beego/validation"

func init() {
	validation.SetDefaultMessage(map[string]string{
		"Required":     "Должно быть заполнено",
		"Min":          "Минимально допустимое значение %d",
		"Max":          "Максимально допустимое значение %d",
		"Range":        "Должно быть в диапазоне от %d до %d",
		"MinSize":      "Минимально допустимая длина %d",
		"MaxSize":      "Максимально допустимая длина %d",
		"Length":       "Длина должна быть равна %d",
		"Alpha":        "Должно состоять из букв",
		"Numeric":      "Должно состоять из цифр",
		"AlphaNumeric": "Должно состоять из букв или цифр",
		"Match":        "Должно совпадать с %s",
		"NoMatch":      "Не должно совпадать с %s",
		"AlphaDash":    "Должно состоять из букв, цифр или символов (-_)",
		"Email":        "Должно быть в правильном формате email",
		"IP":           "Должен быть правильный IP адрес",
		"Base64":       "Должно быть представлено в правильном формате base64",
		"Mobile":       "Должно быть правильным номером мобильного телефона",
		"Tel":          "Должно быть правильным номером телефона",
		"Phone":        "Должно быть правильным номером телефона или мобильного телефона",
		"ZipCode":      "Должно быть правильным почтовым индексом",
	})
}
```

## Examples:

Direct use:

```go
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
    valid.Range(u.Age, 0, 18, "age")

    if valid.HasErrors() {
        // If there are error messages it means the validation didn't pass
        // Print error message
        for _, err := range valid.Errors {
            log.Println(err.Key, err.Message)
        }
    }
    // or use like this
    if v := valid.Max(u.Age, 140, "age"); !v.Ok {
        log.Println(v.Error.Key, v.Error.Message)
    }
    // Customize error messages
    minAge := 18
    valid.Min(u.Age, minAge, "age").Message("18+ only!!")
    // Format error messages
    valid.Min(u.Age, minAge, "age").Message("%d+", minAge)
}
```
Use through StructTag

```go
import (
    "log"
    "strings"

    "github.com/astaxie/beego/validation"
)

// Set validation function in "valid" tag
// Use ";" as the separator of multiple functions. Spaces accept after ";"
// Wrap parameters with "()" and separate parameter with ",". Spaces accept after ","
// Wrap regex match with "//"
// 
// 各个函数的结果的key值为字段名.验证函数名
type user struct {
    Id     int
    Name   string `valid:"Required;Match(/^Bee.*/)"` // Name can't be empty or start with Bee
    Age    int    `valid:"Range(1, 140)"` // 1 <= Age <= 140, only valid in this range
    Email  string `valid:"Email; MaxSize(100)"` // Need to be a valid Email address and no more than 100 characters.
    Mobile string `valid:"Mobile"` // Must be a valid mobile number
    IP     string `valid:"IP"` // Must be a valid IPv4 address
}

// If your struct implemented interface `validation.ValidFormer`
// When all tests in StructTag succeed, it will execute Valid function for custom validation
func (u *user) Valid(v *validation.Validation) {
    if strings.Index(u.Name, "admin") != -1 {
        // Set error messages of Name by SetError and HasErrors will return true
        v.SetError("Name", "Can't contain 'admin' in Name")
    }
}

func main() {
    valid := validation.Validation{}
    u := user{Name: "Beego", Age: 2, Email: "dev@beego.me"}
    b, err := valid.Valid(&u)
    if err != nil {
        // handle error
    }
    if !b {
        // validation does not pass
        // blabla...
        for _, err := range valid.Errors {
            log.Println(err.Key, err.Message)
        }
    }
}
```
Available validation functions in StrucTag:

* `Required` not empty. :TODO 不为空，即各个类型要求不为其零值
* `Min(min int)` minimum value. Valid type is `int`, all other types are invalid.
* `Max(max int)` maximum value. Valid type is `int`, all other types are invalid.
* `Range(min, max int)` Value range. Valid type is `int`, all other types are invalid.
* `MinSize(min int)` minimum length. Valid type is `string slice`, all other types are invalid.
* `MaxSize(max int)` maximum length. Valid type is `string slice`, all other types are invalid.
* `Length(length int)` fixed length. Valid type is `string slice`, all other types are invalid.
* `Alpha` alpha characters. Valid type is `string`, all other types are invalid.
* `Numeric` numerics. Valid type is `string`, all other types are invalid.
* `AlphaNumeric` alpha characters or numerics. Valid type is `string`, all other types are invalid.
* `Match(pattern string)` regex matching. Valid type is `string`, all other types will be cast to string then match. (fmt.Sprintf("%v", obj).Match)
* `AlphaDash` alpha characters or numerics or `-_`. Valid type is `string`, all other types are invalid.
* `Email` Email address. Valid type is `string`, all other types are invalid.
* `IP`  IP address，Only support IPv4 address. Valid type is `string`, all other types are invalid.
* `Base64` base64 encoding. Valid type is `string`, all other types are invalid.
* `Mobile` mobile number. Valid type is `string`, all other types are invalid.
* `Tel` telephone number. Valid type is `string`, all other types are invalid.
* `Phone` mobile number or telephone number. Valid type is `string`, all other types are invalid.
* `ZipCode` zip code. Valid type is `string`, all other types are invalid.

### API doc

Please see [Go Walker](http://gowalker.org/github.com/astaxie/beego/validation)
