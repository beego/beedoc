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
o := NewOrm()
var r RawSeter
r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "slene")
```

* type RawSeter interface {
	* [Exec() (sql.Result, error)](#exec)
	* [QueryRow(...interface{}) error](#queryrow)
	* [QueryRows(...interface{}) (int64, error)](#queryrows)
	* [SetArgs(...interface{}) RawSeter](#setargs)
	* [Values(*[]Params) (int64, error)](#values)
	* [ValuesList(*[]ParamsList) (int64, error)](#valueslist)
	* [ValuesFlat(*ParamsList) (int64, error)](#valuesflat)
	* [Prepare() (RawPreparer, error)](#prepare)
* }

#### Exec

执行sql语句，返回 [sql.Result](http://gowalker.org/database/sql#Result) 对象

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
	Id   int
	Name string
}

var user User
err := o.Raw("SELECT id, name FROM user WHERE id = ?", 1).QueryRow(&user)
```

支持同时 map 多个对象

```go
type User struct {
	Id      int
	Skip    string `orm:"-"`
	SkipYet int
	Name    string
}

var user User
var age int
var created time.Time
o.Raw("SELECT id, NULL, name, age, created FROM user WHERE id = ?", 1).QueryRow(&user, &age, &created)
```

struct 进行 mapper 的时候，可以进行[忽略字段](models.md#忽略字段)，如果有需要临时忽略的 Field 可以在 SELECT 中使用 NULL

支持判断 NULL 对象

```go
type User struct {
	Id   int
	Name string
}

type Profile struct {
	Id   int
	Age  int
}

var user *User
var profile *Profile
err := o.Raw(`SELECT id, name, p.id, p.age FROM user
	LEFT OUTER JOIN profile AS p ON p.id = profile_id WHERE id = ?`, 1).QueryRow(&user, &profile)
if err == nil {
	if profile == nil {
		fmt.Println("user's profile is empty")
	}
}
```

#### QueryRows

QueryRows 支持的对象还有 map 规则是和 QueryRow 一样的，但都是 slice

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

查询多个对象

```go
type Profile struct {
	Id   int
	Age  int
}

var users []*User
var profiles []*Profile
err := o.Raw(`SELECT id, name, p.id, p.age FROM user
	LEFT OUTER JOIN profile AS p ON p.id = profile_id WHERE id = ?`, 1).QueryRows(&users, &profiles)
if err == nil {
	for i, user := range users {
		profile := users[i]
		if profile == nil {
			fmt.Println("user's profile is empty")
		}
	}
}
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

#### Values


返回结果集的 key => value 值

```go
var maps []orm.Params
num, err = o.Raw("SELECT user_name FROM user WHERE status = ?", 1).Values(&maps)
if err == nil && num > 0 {
	fmt.Println(maps[0]["user_name"]) // slene
}
```

#### ValuesList

返回结果集 slice

```go
var lists []orm.ParamsList
num, err = o.Raw("SELECT user_name FROM user WHERE status = ?", 1).ValuesList(&lists)
if err == nil && num > 0 {
	fmt.Println(lists[0][0]) // slene
}
```

#### ValuesFlat

返回单一字段的平铺 slice 数据

```go
var list orm.ParamsList
num, err = o.Raw("SELECT id FROM user WHERE id < ?", 10).ValuesFlat(&list)
if err == nil && num > 0 {
	fmt.Println(list) // []{"1","2","3",...}
}
```

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
