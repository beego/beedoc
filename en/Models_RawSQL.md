# Use SQL statements

* Use Raw SQL query without defining table in ORM.
* Support multiple databases.
* Able to use symbol `?`.
* Support to use Model Struct, Slice and Array.

```go
ids := []int{1, 2, 3}
p.Raw("SELECT name FROM user WHERE id IN (?, ?, ?)", ids)
```

Creat a **RawSeter**:

```go
o := NewOrm()
var r RawSeter
r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "slene")
```

* type RawSeter interface {
	* [Exec() (int64, error)](#exec)
	* [QueryRow(...interface{}) error](#queryrow)
	* [QueryRows(...interface{}) (int64, error)](#queryrows)
	* [SetArgs(...interface{}) RawSeter](#setargs)
	* [Values(*[]Params) (int64, error)](#values)
	* [ValuesList(*[]ParamsList) (int64, error)](#valueslist)
	* [ValuesFlat(*ParamsList) (int64, error)](#valuesflat)
	* [Prepare() (RawPreparer, error)](#prepare)
* }

#### Exec

Execute SQL statements，and returns [sql.Result](http://gowalker.org/database/sql#Result) object:

```go
res, err := o.Raw("UPDATE user SET name = ?", "your").Exec()
if err == nil {
	num, _ := res.RowsAffected()
	fmt.Println("mysql row affected nums: ", num)
}
```
#### QueryRow

QueryRow and QueryRows provide advanced SQL mapper functions:

Support struct:

```go
type User struct {
	Id   int
	Name string
}

var user User
err := o.Raw("SELECT id, name FROM user WHERE id = ?", 1).QueryRow(&user)
```

Support multiple map objects:

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

When struct is using mapper, you're able to [omit fields](Models_Models#omit-fields); use NULL in SELECT to temporary omit field if necessary.

Support check NULL object:

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

QueryRows has same rules of QueryRow except slice:

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

### SetArgs

Change args in Raw(sql, args...) and return a new RawSeter.

This is for a single SQL statement with dynamic arguments:

```go
num, err := r.SetArgs("arg1", "arg2").Exec()
num, err := r.SetArgs("arg1", "arg2").Exec()
...
```

### Values / ValuesList / ValuesFlat

The results of values of Raw SQL are `string` type, NULL fields are represented with empty string.

### Values

Returns key => value result set:

```go
var maps []orm.Params
num, err = o.Raw("SELECT user_name FROM user WHERE status = ?", 1).Values(&maps)
if err == nil && num > 0 {
	fmt.Println(maps[0]["user_name"]) // slene
}
```

### ValuesList

Returns slice of result set:

```go
var lists []orm.ParamsList
num, err = o.Raw("SELECT user_name FROM user WHERE status = ?", 1).ValuesList(&lists)
if err == nil && num > 0 {
	fmt.Println(lists[0][0]) // slene
}
```

### ValuesFlat

Returns the slice of single field:

```go
var list orm.ParamsList
num, err = o.Raw("SELECT id FROM user WHERE id < ?", 10).ValuesList(&list)
if err == nil && num > 0 {
	fmt.Println(list) // []{"1","2","3",...}
}
```

### Prepare

One prepare for multiple exec, to speed up:

```go
p, err := o.Raw("UPDATE user SET name = ? WHERE name = ?").Prepare()
num, err := p.Exec("testing", "slene")
num, err  = p.Exec("testing", "astaxie")
...
...
p.Close() // Do not forget close statement.
```
