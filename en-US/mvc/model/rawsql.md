---
name: Raw SQL to query
sort: 5
---

# Raw SQL to query

* Using Raw SQL to query don't need ORM definition
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
	* [Values(*[]Params) (int64, error)](#values)
	* [ValuesList(*[]ParamsList) (int64, error)](#valueslist)
	* [ValuesFlat(*ParamsList) (int64, error)](#valuesflat)
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

Supports mapping multiple objects at the same time:

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

While mapping struct, it can [ignore fields](models.md#忽略字段). If you just need ignore fields temporary, you can use `NULL` in `SELECT`.

Supports NULL object test

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

Query multiple objects:

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

Changing args param in Raw(sql, args...) can return a new RawSeter.

It can reuse the same SQL query but different params.

```go
res, err := r.SetArgs("arg1", "arg2").Exec()
res, err := r.SetArgs("arg1", "arg2").Exec()
...
```
#### Values / ValuesList / ValuesFlat

The resultSet values returned by Raw SQL query are `string`. NULL field will return empty string ``

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
num, err = o.Raw("SELECT id FROM user WHERE id < ?", 10).ValuesList(&list)
if err == nil && num > 0 {
	fmt.Println(list) // []{"1","2","3",...}
}
```

#### Prepare

Prepare once and exec multiple times to improve the speed of batch execuation.

```go
p, err := o.Raw("UPDATE user SET name = ? WHERE name = ?").Prepare()
res, err := p.Exec("testing", "slene")
res, err  = p.Exec("testing", "astaxie")
...
...
p.Close() // Don't forget to close the prepare.
```
