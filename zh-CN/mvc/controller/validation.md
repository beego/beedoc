---
name: 表单数据验证
sort: 9
---

# 表单验证

表单验证是用于数据验证和错误收集的模块。

## 安装及测试

安装：

	go get github.com/astaxie/beego/validation

测试：

	go test github.com/astaxie/beego/validation

## 示例

直接使用示例：

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
        // 如果有错误信息，证明验证没通过
        // 打印错误信息
        for _, err := range valid.Errors {
            log.Println(err.Key, err.Message)
        }
    }
    // or use like this
    if v := valid.Max(u.Age, 140, "age"); !v.Ok {
        log.Println(v.Error.Key, v.Error.Message)
    }
    // 定制错误信息
    minAge := 18
    valid.Min(u.Age, minAge, "age").Message("少儿不宜！")
    // 错误信息格式化
    valid.Min(u.Age, minAge, "age").Message("%d不禁", minAge)
}
```

通过 StructTag 使用示例：

```go
import (
    "log"
    "strings"

    "github.com/astaxie/beego/validation"
)

// 验证函数写在 "valid" tag 的标签里
// 各个函数之间用分号 ";" 分隔，分号后面可以有空格
// 参数用括号 "()" 括起来，多个参数之间用逗号 "," 分开，逗号后面可以有空格
// 正则函数(Match)的匹配模式用两斜杠 "/" 括起来
// 各个函数的结果的 key 值为字段名.验证函数名
type user struct {
    Id     int
    Name   string `valid:"Required;Match(/^Bee.*/)"` // Name 不能为空并且以 Bee 开头
    Age    int    `valid:"Range(1, 140)"` // 1 <= Age <= 140，超出此范围即为不合法
    Email  string `valid:"Email; MaxSize(100)"` // Email 字段需要符合邮箱格式，并且最大长度不能大于 100 个字符
    Mobile string `valid:"Mobile"` // Mobile 必须为正确的手机号
    IP     string `valid:"IP"` // IP 必须为一个正确的 IPv4 地址
}

// 如果你的 struct 实现了接口 validation.ValidFormer
// 当 StructTag 中的测试都成功时，将会执行 Valid 函数进行自定义验证
func (u *user) Valid(v *validation.Validation) {
    if strings.Index(u.Name, "admin") != -1 {
        // 通过 SetError 设置 Name 的错误信息，HasErrors 将会返回 true
        v.SetError("Name", "名称里不能含有 admin")
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

StructTag 可用的验证函数：

* `Required` 不为空，即各个类型要求不为其零值
* `Min(min int)` 最小值，有效类型：`int`，其他类型都将不能通过验证
* `Max(max int)` 最大值，有效类型：`int`，其他类型都将不能通过验证
* `Range(min, max int)` 数值的范围，有效类型：`int`，他类型都将不能通过验证
* `MinSize(min int)` 最小长度，有效类型：`string slice`，其他类型都将不能通过验证
* `MaxSize(max int)` 最大长度，有效类型：`string slice`，其他类型都将不能通过验证
* `Length(length int)` 指定长度，有效类型：`string slice`，其他类型都将不能通过验证
* `Alpha` alpha字符，有效类型：`string`，其他类型都将不能通过验证
* `Numeric` 数字，有效类型：`string`，其他类型都将不能通过验证
* `AlphaNumeric` alpha 字符或数字，有效类型：`string`，其他类型都将不能通过验证
* `Match(pattern string)` 正则匹配，有效类型：`string`，其他类型都将被转成字符串再匹配(fmt.Sprintf("%v", obj).Match)
* `AlphaDash` alpha 字符或数字或横杠 `-_`，有效类型：`string`，其他类型都将不能通过验证
* `Email` 邮箱格式，有效类型：`string`，其他类型都将不能通过验证
* `IP` IP 格式，目前只支持 IPv4 格式验证，有效类型：`string`，其他类型都将不能通过验证
* `Base64` base64 编码，有效类型：`string`，其他类型都将不能通过验证
* `Mobile` 手机号，有效类型：`string`，其他类型都将不能通过验证
* `Tel` 固定电话号，有效类型：`string`，其他类型都将不能通过验证
* `Phone` 手机号或固定电话号，有效类型：`string`，其他类型都将不能通过验证
* `ZipCode` 邮政编码，有效类型：`string`，其他类型都将不能通过验证

### API 文档

请移步 [Go Walker](http://gowalker.org/github.com/astaxie/beego/validation)。
