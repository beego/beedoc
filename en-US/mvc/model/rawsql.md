---
name: Raw SQL to query
sort: 5
---

# Raw SQL to query

* Using Raw SQL to query doesnt require ORM definition
* Multiple databases support `?` as placeholders and auto convert.
* The params of query support Model Struct, Slice and Array

```go
ids := []int{1, 2, 3}
p.Raw("SELECT name FROM user WHERE id IN (?, ?, ?)", ids)
```

Create a **RawSeter**

```go
o := NewOrm()
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

Run sql query and return [sql.Result](http://gowalker.org/database/sql#Result) object

```go
res, err := o.Raw("UPDATE user SET name = ?", "your").Exec()
if err == nil {
	num, _ := res.RowsAffected()
	fmt.Println("mysql row affected nums: ", num)
}
```

#### QueryRow

QueryRow and QueryRows support high-level sql mapper.

Supports struct:

```go
type User struct {
	Id   int
	Name string
}

var user User
err := o.Raw("SELECT id, name FROM user WHERE id = ?", 1).QueryRow(&user)
```

> from Beego 1.1.0 remove multiple struct support [ISSUE 384](https://github.com/astaxie/beego/issues/384)

#### QueryRows

QueryRows supports the same mapping rules as QueryRow but all of them are slice.

```go
type User struct {
	Id   int
	Name string
}

var users []User
num, err := o.Raw("SELECT id, name FROM user WHERE id = ?", 1).QueryRows(&users)
if err == nil {
	fmt.Println("user nums: ", num)
}
```

> from Beego 1.1.0 remove multiple struct support [ISSUE 384](https://github.com/astaxie/beego/issues/384)

#### SetArgs

Changing args param in Raw(sql, args...) can return a new RawSeter.

It can reuse the same SQL query but different params.

```go
res, err := r.SetArgs("arg1", "arg2").Exec()
res, err := r.SetArgs("arg1", "arg2").Exec()
...
```
#### Values / ValuesList / ValuesFlat

The resultSet values returned by Raw SQL query are `string`. NULL field will return empty string ``

> from Beego 1.1.0 
> Values, ValuesList, ValuesFlat. The returned fields can be specified.
> Generally you don't need to specify. Because the field names are already defined in your SQL.

#### Values

The key => value pairs of resultSet:

```go
var maps []orm.Params
num, err = o.Raw("SELECT user_name FROM user WHERE status = ?", 1).Values(&maps)
if err == nil && num > 0 {
	fmt.Println(maps[0]["user_name"]) // slene
}
```

#### ValuesList

slice of resultSet

```go
var lists []orm.ParamsList
num, err = o.Raw("SELECT user_name FROM user WHERE status = ?", 1).ValuesList(&lists)
if err == nil && num > 0 {
	fmt.Println(lists[0][0]) // slene
}
```

#### ValuesFlat

Return slice of a single field:

```go
var list orm.ParamsList
num, err = o.Raw("SELECT id FROM user WHERE id < ?", 10).ValuesFlat(&list)
if err == nil && num > 0 {
	fmt.Println(list) // []{"1","2","3",...}
}
```

#### RowsToMap

SQL query results

| name | value |
| --- | --- |
| total | 100 |
| found | 200 |

map rows results to map

```go
res := make(orm.Params)
nums, err := o.Raw("SELECT name, value FROM options_table").RowsToMap(&res, "name", "value")
// res is a map[string]interface{}{
//	"total": 100,
//	"found": 200,
// }
```

#### RowsToStruct

SQL query results

| name | value |
| --- | --- |
| total | 100 |
| found | 200 |

map rows results to struct

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

> support name conversion: snake -> camel, eg: SELECT user_name ... to your struct field UserName.

#### Prepare

Prepare once and exec multiple times to improve the speed of batch execution.

```go
p, err := o.Raw("UPDATE user SET name = ? WHERE name = ?").Prepare()
res, err := p.Exec("testing", "slene")
res, err  = p.Exec("testing", "astaxie")
...
...
p.Close() // Don't forget to close the prepare.
```
