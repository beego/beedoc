---
name: 原生 SQL 查询
sort: 5
---

# 使用SQL语句进行查询

* 使用 Raw SQL 查询，无需使用 ORM 表定义
* 多数据库，都可直接使用占位符号 `?`，自动转换
* 查询时的参数，支持使用 Model Struct 和 Slice, Array

```go
ids := []int{1, 2, 3}
p.Raw("SELECT name FROM user WHERE id IN (?, ?, ?)", ids)
```

创建一个 **RawSeter**

```go
o := orm.NewOrm()
var r RawSeter
r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "slene")
```

* type RawSeter interface {
	* [Exec() (sql.Result, error)](#exec)
	* [QueryRow(...interface{}) error](#queryrow)
	* [QueryRows(...interface{}) (int64, error)](#queryrows)
	* [SetArgs(...interface{}) RawSeter](#setargs)
	* [Values(*[]Params, ...string) (int64, error)](#values)
	* [ValuesList(*[]ParamsList, ...string) (int64, error)](#valueslist)
	* [ValuesFlat(*ParamsList, string) (int64, error)](#valuesflat)
	* [RowsToMap(*Params, string, string) (int64, error)](#rowstomap)
	* [RowsToStruct(interface{}, string, string) (int64, error)](#rowstostruct)
	* [Prepare() (RawPreparer, error)](#prepare)
* }

#### Exec

执行 sql 语句，返回 [sql.Result](http://gowalker.org/database/sql#Result) 对象

```go
res, err := o.Raw("UPDATE user SET name = ?", "your").Exec()
if err == nil {
	num, _ := res.RowsAffected()
	fmt.Println("mysql row affected nums: ", num)
}
```

#### QueryRow

QueryRow 和 QueryRows 提供高级 sql mapper 功能

支持 struct

```go
type User struct {
	Id       int
	UserName string
}

var user User
err := o.Raw("SELECT id, user_name FROM user WHERE id = ?", 1).QueryRow(&user)
```

> from beego 1.1.0 取消了多个对象支持 [ISSUE 384](https://github.com/astaxie/beego/issues/384)

#### QueryRows

QueryRows 支持的对象还有 map 规则是和 QueryRow 一样的，但都是 slice

```go
type User struct {
	Id       int
	UserName string
}

var users []User
num, err := o.Raw("SELECT id, user_name FROM user WHERE id = ?", 1).QueryRows(&users)
if err == nil {
	fmt.Println("user nums: ", num)
}
```

> from beego 1.1.0 取消了多个对象支持 [ISSUE 384](https://github.com/astaxie/beego/issues/384)
```

#### SetArgs

改变 Raw(sql, args...) 中的 args 参数，返回一个新的 RawSeter

用于单条 sql 语句，重复利用，替换参数然后执行。

```go
res, err := r.SetArgs("arg1", "arg2").Exec()
res, err := r.SetArgs("arg1", "arg2").Exec()
...
```

#### Values / ValuesList / ValuesFlat

Raw SQL 查询获得的结果集 Value 为 `string` 类型，NULL 字段的值为空 ``

> from beego 1.1.0
> Values, ValuesList, ValuesFlat 的参数，可以指定返回哪些 Columns 的数据
> 通常情况下，是无需指定的，因为 sql 语句中你可以自行设置 SELECT 的字段

#### Values

返回结果集的 key => value 值

```go
var maps []orm.Params
num, err := o.Raw("SELECT user_name FROM user WHERE status = ?", 1).Values(&maps)
if err == nil && num > 0 {
	fmt.Println(maps[0]["user_name"]) // slene
}
```

#### ValuesList

返回结果集 slice

```go
var lists []orm.ParamsList
num, err := o.Raw("SELECT user_name FROM user WHERE status = ?", 1).ValuesList(&lists)
if err == nil && num > 0 {
	fmt.Println(lists[0][0]) // slene
}
```

#### ValuesFlat

返回单一字段的平铺 slice 数据

```go
var list orm.ParamsList
num, err := o.Raw("SELECT id FROM user WHERE id < ?", 10).ValuesFlat(&list)
if err == nil && num > 0 {
	fmt.Println(list) // []{"1","2","3",...}
}
```

#### RowsToMap

SQL 查询结果是这样

| name | value |
| --- | --- |
| total | 100 |
| found | 200 |

查询结果匹配到 map 里

```go
res := make(orm.Params)
nums, err := o.Raw("SELECT name, value FROM options_table").RowsToMap(&res, "name", "value")
// res is a map[string]interface{}{
//	"total": 100,
//	"found": 200,
// }
```

#### RowsToStruct

SQL 查询结果是这样

| name | value |
| --- | --- |
| total | 100 |
| found | 200 |

查询结果匹配到 struct 里

```go
type Options struct {
	Total int
	Found int
}

res := new(Options)
nums, err := o.Raw("SELECT name, value FROM options_table").RowsToStruct(res, "name", "value")
fmt.Println(res.Total) // 100
fmt.Println(res.Found) // 200
```

> 匹配支持的名称转换为 snake -> camel, eg: SELECT user_name ... 需要你的 struct 中定义有 UserName

#### Prepare

用于一次 prepare 多次 exec，以提高批量执行的速度。

```go
p, err := o.Raw("UPDATE user SET name = ? WHERE name = ?").Prepare()
res, err := p.Exec("testing", "slene")
res, err  = p.Exec("testing", "astaxie")
...
...
p.Close() // 别忘记关闭 statement
```
