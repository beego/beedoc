# Advance query

ORM uses **QuerySeter** to organize query, every method that returns  **QuerySeter** will give you a new **QuerySeter** object.

Basic usage:

```go
o := orm.NewOrm()

// Get QuerySeter object, table name is user.
qs := o.QueryTable("user")

// Or use object as table name.
user := new(User)
qs = o.QueryTable(user) // 返回 QuerySeter
```

## expr

expr describes fields and SQL operators in `QuerySeter`.

Fields organized according to relation of tables. For example, `User` has a outside key `Profile`, so if you want to use `Profile.Age` as the condition, you have to the expression `Profile__Age`. Note that the saperator is double under lines `__`. You can append operations of SQL to expr except you use expr to describe fields. For example, `Profile__Age__gt` represents condition query `Profile.Age > 18`.

Comments describes SQL statements that similar to the expr, not the exactly generated results.

```go
qs.Filter("id", 1) // WHERE id = 1
qs.Filter("profile__age", 18) // WHERE profile.age = 18
qs.Filter("Profile__Age", 18) // Both table and struct field name are acceptable.
qs.Filter("profile__age", 18) // WHERE profile.age = 18
qs.Filter("profile__age__gt", 18) // WHERE profile.age > 18
qs.Filter("profile__age__gte", 18) // WHERE profile.age >= 18
qs.Filter("profile__age__in", 18, 20) // WHERE profile.age IN (18, 20)

qs.Filter("profile__age__in", 18, 20).Exclude("profile__lt", 1000)
// WHERE profile.age IN (18, 20) AND NOT profile_id < 1000
```

## Operators

Currently supported operators:

* [exact](#exact) / [iexact](#iexact) equals
* [contains](#contains) / [icontains](#icontains) contains
* [gt / gte](#gt / gte) greater / greater or equal
* [lt / lte](#lt / lte) less / less or equal
* [startswith](#startswith) / [istartswith](#istartswith) starts with
* [endswith](#endswith) / [iendswith](#iendswith) ends with
* [in](#in)
* [isnull](#isnull)

Starts with `i` means that's not case sensitive.

### exact

Default values of Filter / Exclude / Condition expr:

```go
qs.Filter("name", "slene") // WHERE name = 'slene'
qs.Filter("name__exact", "slene") // WHERE name = 'slene'
// Use `=` to match, data table collation decides whether it's case sensitive or not.
qs.Filter("profile", nil) // WHERE profile_id IS NULL
```

### iexact

```go
qs.Filter("name__iexact", "slene")
// WHERE name LIKE 'slene'
// Not case sensitive, 'Slene' 'sLENE' are both acceptable.
```

### contains

```go
qs.Filter("name__contains", "slene")
// WHERE name LIKE BINARY '%slene%'
// Case sensitive, matches all characters that contain "slene".
```

### icontains

```go
qs.Filter("name__icontains", "slene")
// WHERE name LIKE '%slene%'
// Not case sensitive, 'im Slene', 'im sLENE' are both acceptable.
```

### in

```go
qs.Filter("profile__age__in", 17, 18, 19, 20)
// WHERE profile.age IN (17, 18, 19, 20)
```

### gt / gte

```go
qs.Filter("profile__age__gt", 17)
// WHERE profile.age > 17

qs.Filter("profile__age__gte", 18)
// WHERE profile.age >= 18
```

### lt / lte

```go
qs.Filter("profile__age__lt", 17)
// WHERE profile.age < 17

qs.Filter("profile__age__lte", 18)
// WHERE profile.age <= 18
```

### startswith

```go
qs.Filter("name__startswith", "slene")
// WHERE name LIKE BINARY 'slene%'
// Case sensitive, matches all strings that start with 'slene'.
```

### istartswith

```go
qs.Filter("name__istartswith", "slene")
// WHERE name LIKE 'slene%'
// Not case sensitive, matches all strings start with something like 'slene', 'Slene'.
```

### endswith

```go
qs.Filter("name__endswith", "slene")
// WHERE name LIKE BINARY '%slene'
// Case sensitive, matches all strings that end with 'slene'.
```

### iendswith

```go
qs.Filter("name__startswith", "slene")
// WHERE name LIKE '%slene'
// Not case sensitive, matches all strings end with something like 'slene', 'Slene'.
```

### isnull

```go
qs.Filter("profile__isnull", true)
qs.Filter("profile_id__isnull", true)
// WHERE profile_id IS NULL

qs.Filter("profile__isnull", false)
// WHERE profile_id IS NOT NULL
```

## Advanced query interface

QuerySeter is the advanced query interface, let's talk a look:

* type QuerySeter interface {
	* [Filter(string, ...interface{}) QuerySeter](#filter)
	* [Exclude(string, ...interface{}) QuerySeter](#exclude)
	* [SetCond(*Condition) QuerySeter](#setcond)
	* [Limit(int, ...int64) QuerySeter](#limit)
	* [Offset(int64) QuerySeter](#offset)
	* [OrderBy(...string) QuerySeter](#orderby)
	* [RelatedSel(...interface{}) QuerySeter](#relatedsel)
	* [Count() (int64, error)](#count)
	* [Update(Params) (int64, error)](#update)
	* [Delete() (int64, error)](#delete)
	* [PrepareInsert() (Inserter, error)](#prepareinsert)
	* [All(interface{}) (int64, error)](#all)
	* [One(Modeler) error](#one)
	* [Values(*[]Params, ...string) (int64, error)](#values)
	* [ValuesList(*[]ParamsList, ...string) (int64, error)](#valueslist)
	* [ValuesFlat(*ParamsList, string) (int64, error)](#valuesflat)
* }

* Every API that returns `QuerySeter` will returns a new `QuerySeter` that no relationship with the old one.

* Use `Filter` and `Exclude` to do commonly conditional query which contains two kinds of filter rules: contain and exclude. 

### Filter

Contain filter example, use `AND` to connect multiple filters:

```go
qs.Filter("profile__isnull", true).Filter("name", "slene")
// WHERE profile_id IS NULL AND name = 'slene'
```

### Exclude

Exclude filter example, use `NOT` to exclude conditions, and `AND` to connect multiple Excludes:

```go
qs.Exclude("profile__isnull", true).Filter("name", "slene")
// WHERE NOT profile_id IS NULL AND name = 'slene'
```

### SetCond

Customized conditional expression:

```go
cond := NewCondition()
cond1 := cond.And("profile__isnull", false).AndNot("status__in", 1).Or("profile__age__gt", 2000)

qs := orm.QueryTable("user")
qs = qs.SetCond(cond1)
// WHERE ... AND ... AND NOT ... OR ...

cond2 := cond.AndCond(cond1).OrCond(cond.And("name", "slene"))
qs = qs.SetCond(cond2).Count()
// WHERE (... AND ... AND NOT ... OR ...) OR ( ... )
```

### Limit

Limit maximum returns rows, the 2nd argument is for setting `Offset`.

```go
var DefaultRowsLimit = 1000 // Default limitation of ORM is 1000.

qs.Limit(10)
// LIMIT 10

qs.Limit(10, 20)
// LIMIT 10 OFFSET 20

qs.Limit(-1)
// no limit

qs.Limit(-1, 100)
// LIMIT 18446744073709551615 OFFSET 100
// 18446744073709551615 is 1<<64 - 1 in case that no limitation of return rows but has offset.
```

### Offset
	
Set offset rows:

```go
qs.Offset(20)
// LIMIT 1000 OFFSET 20
```

### OrderBy

Arguments are **expr**:

Use minus `-` to represent `DESC`:

```go
qs.OrderBy("id", "-profile__age")
// ORDER BY id ASC, profile.age DESC

qs.OrderBy("-profile__age", "profile")
// ORDER BY profile.age DESC, profile_id ASC
```

### RelatedSel

Arguments are **expr**:

```go
var DefaultRelsDepth = 5 // `RelatedSel` will try 5 levels relational query as default.

qs := o.QueryTable("post")

qs.RelateSel()
// INNER JOIN user ... LEFT OUTER JOIN profile ...

qs.RelateSel("user")
// INNER JOIN user ... 
// Use expr to set fields that need to do relational query.

// Use `LEFT OUTER JOIN` for fields that have null properties.
```

### Count

Total rows of results:

```go
cnt, err := o.QueryTable("user").Count() // SELECT COUNT(*) FROM USER
fmt.Printf("Count Num: %s, %s", cnt, err)
```

### Update

Batch update operation:

```go
num, err := o.QueryTable("user").Filter("name", "slene").Update(orm.Params{
	"name": "astaxie",
})
fmt.Printf("Affected Num: %s, %s", num, err)
// SET name = "astaixe" WHERE name = "slene"
```

### Delete

Batch delete operation:

```go
num, err := o.QueryTable("user").Filter("name", "slene").Delete()
fmt.Printf("Affected Num: %s, %s", num, err)
// DELETE FROM user WHERE name = "slene"
```

### PrepareInsert

Execute `insert` many times in one `prepare` to speed up:

```go
var users []*User
...
qs := o.QueryTable("user")
i, _ := qs.PrepareInsert()
for _, user := range users {
	id, err := i.Insert(user)
	if err != nil {
		...
	}
}
// PREPARE INSERT INTO user (`name`, ...) VALUES (?, ...)
// EXECUTE INSERT INTO user (`name`, ...) VALUES ("slene", ...)
// EXECUTE ...
// ...
i.Close() // 别忘记关闭 statement
```

### All

Returns corresponding object result set: 

```go
var users []User
num, err := o.QueryTable("user").Filter("name", "slene").All(&users)
fmt.Printf("Returned Rows Num: %s, %s", num, err)
```

### One

Try to return single record:

```go
var user User
err := o.QueryTable("user").Filter("name", "slene").One(&user)
if err == orm.ErrMultiRows {
	fmt.Printf("Returned Multi Rows Not One")
}
if err == orm.ErrNoRows {
	fmt.Printf("Not row found")
}
```

### Values

Returns key => value of result set.

Key is the field name of the Model, values saved as string:

```go
var maps []orm.Params
num, err := o.QueryTable("user").Values(&maps)
if err != nil {
	fmt.Printf("Result Nums: %d\n", num)
	for _, m := range maps {
		fmt.Println(m["Id"], m["Name"])
	}
}
```

Returns specific field's data:

```go
var maps []orm.Params
num, err := o.QueryTable("user").Values(&maps, "id", "name", "profile", "profile__age")
if err != nil {
	fmt.Printf("Result Nums: %d\n", num)
	for _, m := range maps {
		fmt.Println(m["Id"], m["Name"], m["Profile"], m["Profile__Age"])
		// Data in map are expanded, no nested.
	}
}
```

### ValuesList

Results are saved in slice, the order of results is the same order of fields in Model, elements are saved as string:

```go
var lists []orm.ParamsList
num, err := o.QueryTable("user").ValuesList(&lists)
if err != nil {
	fmt.Printf("Result Nums: %d\n", num)
	for _, row := range lists {
		fmt.Println(row)
	}
}
```

You can use expr to indicate which field to be returned:

```go
var lists []orm.ParamsList
num, err := o.QueryTable("user").ValuesList(&lists, "name", "profile__age")
if err != nil {
	fmt.Printf("Result Nums: %d\n", num)
	for _, row := range lists {
		fmt.Printf("Name: %s, Age: %s\m", row[0], row[1])
	}
}
```

### ValuesFlat

Only returned specific field value into single slice:

```go
var list orm.ParamsList
num, err := o.QueryTable("user").ValuesFlat(&list, "name")
if err != nil {
	fmt.Printf("Result Nums: %d\n", num)
	fmt.Printf("All User Names: %s", strings.Join(list, ", ")
}
```

## Relation query

See example in [Model_ORM](Models_ORM) for how.

#### The relation between User and Profile is OneToOne

Already had `User` object, to query `Profile`:

```go
user := &User{Id: 1}
o.Read(user)
if user.Profile != nil {
	o.Read(user.Profile)
}
```

Directly query:

```go
user := &User{}
o.QueryTable("user").Filter("Id", 1).RelatedSel().One(user)
// Auto-find Profile
fmt.Println(user.Profile)
// Because reserve relation in Profile for User, User that in Profile will be auto-assign as well and be directly used.
fmt.Println(user.Profile.User)
```

Through User to reserve query Profile：

```go
var profile Profile
err := o.QueryTable("profile").Filter("User__Id", 1).One(&profile)
if err == nil {
	fmt.Println(profile)
}
```

#### The relation between Post and User is ManyToOne, which means ForeignKey is User

```go
type Post struct {
	Id    int
	Title string
	User  *User  `orm:"rel(fk)"`
	Tags  []*Tag `orm:"rel(m2m)"`
}
```

```go
var posts []*Post
num, err := o.QueryTable("post").Filter("User", 1).RelatedSel().All(&posts)
if err == nil {
	fmt.Printf("%d posts read\n", num)
	for _, post := range posts {
		fmt.Printf("Id: %d, UserName: %d, Title: %s\n", post.Id, post.User.UserName, post.Title)
	}
}
```

According to Post.Title query corresponding User：

When you do RegisterModel, ORM also auto-create reserve relation of Post in User, so you are able to directly query:

```go
var user User
err := o.QueryTable("user").Filter("Post__Title", "The Title").Limit(1).One(&user)
if err == nil {
	fmt.Printf(user)
}
```

#### The relation between Post and Tag is ManyToMany

```go
type Tag struct {
	Id    int
	Name  string
}
```
TODO
