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

### Exec

Execute SQL statements:

```go
num, err := r.Exec()
```

### QueryRow

TODO

### QueryRows

TODO

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
