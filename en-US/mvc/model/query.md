---
name: Advanced Queries
sort: 4
---

# Advanced Queries

ORM uses **QuerySeter** to organize query, every method that returns  **QuerySeter** will give you a new **QuerySeter** object.


Basic Usage:

```go
o := orm.NewOrm()

// Get a QuerySeter object. User is table name
qs := o.QueryTable("user")

// Can also use object as table name
user := new(User)
qs = o.QueryTable(user) // return a QuerySeter
```
## expr


expr describes fields and SQL operators in `QuerySeter`.

Field combination orders are decided by the relationship of tables. For example, `User` has a foreign key to `Profile`, so if you want to use `Profile.Age` as the condition, you have to use the expression `Profile__Age`. Note that the separator is double under scores `__`. `Expr` can also append operators at the end to execute related SQL. For example, `Profile__Age__gt` represents condition query `Profile.Age > 18`.

Comments below describe SQL statements that are similar to the expr, not the exactly generated results.

```go
qs.Filter("id", 1) // WHERE id = 1
qs.Filter("profile__age", 18) // WHERE profile.age = 18
qs.Filter("Profile__Age", 18) // key name and field name are both valid
qs.Filter("profile__age", 18) // WHERE profile.age = 18
qs.Filter("profile__age__gt", 18) // WHERE profile.age > 18
qs.Filter("profile__age__gte", 18) // WHERE profile.age >= 18
qs.Filter("profile__age__in", 18, 20) // WHERE profile.age IN (18, 20)

qs.Filter("profile__age__in", 18, 20).Exclude("profile__lt", 1000)
// WHERE profile.age IN (18, 20) AND NOT profile_id < 1000
```
## Operators

The supported operators:

* [exact](#exact) / [iexact](#iexact) equal to
* [contains](#contains) / [icontains](#icontains) contains
* [gt / gte](#gt / gte) greater than / greater than or equal to
* [lt / lte](#lt / lte) less than / less than or equal to
* [startswith](#startswith) / [istartswith](#istartswith) starts with
* [endswith](#endswith) / [iendswith](#iendswith) ends with
* [in](#in)
* [isnull](#isnull)

The operators that start with `i` ignore case.

### exact

Default values of Filter, Exclude and Condition expr 

```go
qs.Filter("name", "slene") // WHERE name = 'slene'
qs.Filter("name__exact", "slene") // WHERE name = 'slene'
// using = , case sensitive or not is depending on which collation database table is used
qs.Filter("profile", nil) // WHERE profile_id IS NULL
```

### iexact

```go
qs.Filter("name__iexact", "slene")
// WHERE name LIKE 'slene'
// Case insensitive, will match any name that equals to 'slene'
```

### contains

```go
qs.Filter("name__contains", "slene")
// WHERE name LIKE BINARY '%slene%'
// Case sensitive, only match name that contains 'slene'
```

### icontains

```go
qs.Filter("name__icontains", "slene")
// WHERE name LIKE '%slene%'
// Case insensitive, will match any name that contains 'slene'
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
// Case sensitive, only match name that starts with 'slene'
```

### istartswith

```go
qs.Filter("name__istartswith", "slene")
// WHERE name LIKE 'slene%'
// Case insensitive, will match any name that starts with 'slene'
```

### endswith

```go
qs.Filter("name__endswith", "slene")
// WHERE name LIKE BINARY '%slene'
// Case sensitive, only match name that ends with 'slene'
```

### iendswith

```go
qs.Filter("name__iendswith", "slene")
// WHERE name LIKE '%slene'
// Case insensitive, will match any name that ends with 'slene'
```

### isnull

```go
qs.Filter("profile__isnull", true)
qs.Filter("profile_id__isnull", true)
// WHERE profile_id IS NULL

qs.Filter("profile__isnull", false)
// WHERE profile_id IS NOT NULL
```

## Advanced Query API

QuerySeter is the API of advanced query. Here are its methods:

* type QuerySeter interface {
	* [Filter(string, ...interface{}) QuerySeter](#filter)
	* [Exclude(string, ...interface{}) QuerySeter](#exclude)
	* [SetCond(*Condition) QuerySeter](#setcond)
	* [Limit(int, ...int64) QuerySeter](#limit)
	* [Offset(int64) QuerySeter](#offset)
	* [GroupBy(...string) QuerySeter](#groupby)
	* [OrderBy(...string) QuerySeter](#orderby)
	* [Distinct() QuerySeter](#distinct)
	* [RelatedSel(...interface{}) QuerySeter](#relatedsel)
	* [Count() (int64, error)](#count)
	* [Exist() bool](#exist)
	* [Update(Params) (int64, error)](#update)
	* [Delete() (int64, error)](#delete)
	* [PrepareInsert() (Inserter, error)](#prepareinsert)
	* [All(interface{}, ...string) (int64, error)](#all)
	* [One(interface{}, ...string) error](#one)
	* [Values(*[]Params, ...string) (int64, error)](#values)
	* [ValuesList(*[]ParamsList, ...string) (int64, error)](#valueslist)
	* [ValuesFlat(*ParamsList, string) (int64, error)](#valuesflat)
* }

* Every API call that returns **QuerySeter** will give you a new **QuerySeter** object. It won't affect the previous object

* Advanced query uses `Filter` and `Exclude` to do conditional queries.  There are two filter rules: contain and exclude

### Filter

Used to filter the result for the **include conditions**.

Use `AND` to connect multiple filters:

```go
qs.Filter("profile__isnull", true).Filter("name", "slene")
// WHERE profile_id IS NULL AND name = 'slene'
```

### Exclude

Used to filter the result for the **exclude conditions**.

Use `NOT` to exclude condition
Use `AND` to connect multiple filters:

```go
qs.Exclude("profile__isnull", true).Filter("name", "slene")
// WHERE NOT profile_id IS NULL AND name = 'slene'
```

### SetCond

Custom conditions:

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

Limit maximum returned lines. The second param can set `Offset`

```go
var DefaultRowsLimit = 1000 // The default limit of ORM is 1000

// LIMIT 1000

qs.Limit(10)
// LIMIT 10

qs.Limit(10, 20)
// LIMIT 10 OFFSET 20

qs.Limit(-1)
// no limit

qs.Limit(-1, 100)
// LIMIT 18446744073709551615 OFFSET 100
// 18446744073709551615 is 1<<64 - 1. Used to set the condition which is no limit but with offset
```

### Offset
	
Set offset lines:

```go
qs.Offset(20)
// LIMIT 1000 OFFSET 20
```

### GroupBy

```go
qs.GroupBy("id", "age")
// GROUP BY id,age
```

### OrderBy

Param uses **expr**

Using `-` at the beginning of expr stands for order by `DESC`

```go
qs.OrderBy("id", "-profile__age")
// ORDER BY id ASC, profile.age DESC

qs.OrderBy("-profile__age", "profile")
// ORDER BY profile.age DESC, profile_id ASC
```

### Distinct
	
Same as `distinct` statement in sql, return only distinct (different) values

```go
qs.Distinct()
// SELECT DISTINCT
```

### RelatedSel

Relational queries. Param uses **expr**

```go
var DefaultRelsDepth = 5 // RelatedSel will query for maximum 5 level by default

qs := o.QueryTable("post")

qs.RelatedSel()
// INNER JOIN user ... LEFT OUTER JOIN profile ...

qs.RelatedSel("user")
// INNER JOIN user ... 
// Only query the fields set by expr

// For fields with null attribute will use LEFT OUTER JOIN
```

### Count

Return line count based on the current query

```go
cnt, err := o.QueryTable("user").Count() // SELECT COUNT(*) FROM USER
fmt.Printf("Count Num: %s, %s", cnt, err)
```

### Exist

```go
exist := o.QueryTable("user").Filter("UserName", "Name").Exist()
fmt.Printf("Is Exist: %s", exist)
```

### Update

Execute batch updating based on the current query

```go
num, err := o.QueryTable("user").Filter("name", "slene").Update(orm.Params{
	"name": "astaxie",
})
fmt.Printf("Affected Num: %s, %s", num, err)
// SET name = "astaixe" WHERE name = "slene"
```

Atom operation add field:

```go
// Assume there is a nums int field in user struct
num, err := o.QueryTable("user").Update(orm.Params{
	"nums": orm.ColValue(orm.Col_Add, 100),
})
// SET nums = nums + 100
```

orm.ColValue supports:

```go
Col_Add      // plus
Col_Minus    // minus 
Col_Multiply // multiply 
Col_Except   // divide
```

### Delete

Execute batch deletion based on the current query

```go
num, err := o.QueryTable("user").Filter("name", "slene").Delete()
fmt.Printf("Affected Num: %s, %s", num, err)
// DELETE FROM user WHERE name = "slene"
```

### PrepareInsert

Use a prepared statement to increase inserting speed with multiple inserts.

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
i.Close() // Don't forget to close the statement
```

### All

Return the related ResultSet

Param of `All` supports *[]Type and *[]*Type

```go
var users []*User
num, err := o.QueryTable("user").Filter("name", "slene").All(&users)
fmt.Printf("Returned Rows Num: %s, %s", num, err)
```

All / Values / ValuesList / ValuesFlat will be limited by [Limit](#limit). 1000 lines by default.

The returned fields can be specified:

```go
type Post struct {
	Id      int
	Title   string
	Content string
	Status  int
}

// Only return Id and Title
var posts []Post
o.QueryTable("post").Filter("Status", 1).All(&posts, "Id", "Title")
```

The other fields of the object are set to the default value of the field's type.

### One

Try to return one record

```go
var user User
err := o.QueryTable("user").Filter("name", "slene").One(&user)
if err == orm.ErrMultiRows {
	// Have multiple records
	fmt.Printf("Returned Multi Rows Not One")
}
if err == orm.ErrNoRows {
	// No result 
	fmt.Printf("Not row found")
}
```

The returned fields can be specified:

```go
// Only return Id and Title
var post Post
o.QueryTable("post").Filter("Content__istartswith", "prefix string").One(&post, "Id", "Title")
```

The other fields of the object are set to the default value of the fields' type.

### Values

Return key => value of result set

key is Field name in Model. value type if string.

```go
var maps []orm.Params
num, err := o.QueryTable("user").Values(&maps)
if err == nil {
	fmt.Printf("Result Nums: %d\n", num)
	for _, m := range maps {
		fmt.Println(m["Id"], m["Name"])
	}
}
```

Return specific fields:

**TODO**: doesn't support recursive query. **RelatedSel** return Values directly

But it can specify the value needed by expr.

```go
var maps []orm.Params
num, err := o.QueryTable("user").Values(&maps, "id", "name", "profile", "profile__age")
if err == nil {
	fmt.Printf("Result Nums: %d\n", num)
	for _, m := range maps {
		fmt.Println(m["Id"], m["Name"], m["Profile"], m["Profile__Age"])
    // There is no complicated nesting data in the map
	}
}
```

### ValuesList

The result set will be stored as a slice

The order of the result is same as the Fields order in the Model definition.

The values are saved as strings.

```go
var lists []orm.ParamsList
num, err := o.QueryTable("user").ValuesList(&lists)
if err == nil {
	fmt.Printf("Result Nums: %d\n", num)
	for _, row := range lists {
		fmt.Println(row)
	}
}
```

It can return specific fields by setting expr.

```go
var lists []orm.ParamsList
num, err := o.QueryTable("user").ValuesList(&lists, "name", "profile__age")
if err == nil {
	fmt.Printf("Result Nums: %d\n", num)
	for _, row := range lists {
		fmt.Printf("Name: %s, Age: %s\m", row[0], row[1])
	}
}
```

### ValuesFlat

Only returns a single values slice of a specific field.

```go
var list orm.ParamsList
num, err := o.QueryTable("user").ValuesFlat(&list, "name")
if err == nil {
	fmt.Printf("Result Nums: %d\n", num)
	fmt.Printf("All User Names: %s", strings.Join(list, ", ")
}
```

## Relational Query

Let's see how to do Relational Query by looking at [Model Definition](orm.md)

#### User and Profile is OnToOne relation

Query Profile by known User object:

```go
user := &User{Id: 1}
o.Read(user)
if user.Profile != nil {
	o.Read(user.Profile)
}
```

Cascaded query directly:

```go
user := &User{}
o.QueryTable("user").Filter("Id", 1).RelatedSel().One(user)
// Get Profile automatically
fmt.Println(user.Profile)
// Because In Profile we defined reverse relation User, Profile's User is also auto assigned. Can directly use:
fmt.Println(user.Profile.User)
```
Reverse finding Profile by User:

```go
var profile Profile
err := o.QueryTable("profile").Filter("User__Id", 1).One(&profile)
if err == nil {
	fmt.Println(profile)
}
```

#### Post and User are ManyToOne relation. i.e.: ForeignKey is User

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

Query related User by Post.Title:

While RegisterModel, ORM will create reverse relation for Post in User. So it can query directly:

```go
var user User
err := o.QueryTable("user").Filter("Post__Title", "The Title").Limit(1).One(&user)
if err == nil {
	fmt.Printf(user)
}
```

#### Post and Tag are ManyToMany relation

After setting rel(m2m), ORM will create connecting table automatically.

```go
type Post struct {
	Id    int
	Title string
	User  *User  `orm:"rel(fk)"`
	Tags  []*Tag `orm:"rel(m2m)"`
}
```

```go
type Tag struct {
	Id    int
	Name  string
	Posts []*Post `orm:"reverse(many)"`
}
```

Query which post used the tag with tag name:

```go
var posts []*Post
num, err := dORM.QueryTable("post").Filter("Tags__Tag__Name", "golang").All(&posts)
```

Query how many tags does the post have with post title:

```go
var tags []*Tag
num, err := dORM.QueryTable("tag").Filter("Posts__Post__Title", "Introduce Beego ORM").All(&tags)
```

## Load Related Field

LoadRelated is used to load relation field of model. Including all rel/reverse - one/many relation.

Load ManyToMany relation field

```go
// load related Tags
post := Post{Id: 1}
err := o.Read(&post)
num, err := o.LoadRelated(&post, "Tags")
```

```go
// Load related Posts
tag := Tag{Id: 1}
err := o.Read(&tag)
num, err := o.LoadRelated(&tag, "Posts")
```

User is the ForeignKey of Post. Load related ReverseMany

```go
type User struct {
	Id    int
	Name  string
	Posts []*Post `orm:"reverse(many)"`
}

user := User{Id: 1}
err := dORM.Read(&user)
num, err := dORM.LoadRelated(&user, "Posts")
for _, post := range user.Posts {
	//...
}
```

## Handling ManyToMany relation

* type QueryM2Mer interface {
	* [Add(...interface{}) (int64, error)](#querym2mer-add)
	* [Remove(...interface{}) (int64, error)](#querym2mer-remove)
	* [Exist(interface{}) bool](#querym2mer-exist)
	* [Clear() (int64, error)](#querym2mer-clear)
	* [Count() (int64, error)](#querym2mer-count)
* }

Create a QueryM2Mer object

```go
o := orm.NewOrm()
post := Post{Id: 1}
m2m := o.QueryM2M(&post, "Tags")
// In the first param object must have primary key
// The second param is the M2M field will work with
// API of QueryM2Mer will used to Post with id equals 1
```

### QueryM2Mer Add

```go
tag := &Tag{Name: "golang"}
o.Insert(tag)

num, err := m2m.Add(tag)
if err == nil {
	fmt.Println("Added nums: ", num)
}
```

Add supports many types: Tag *Tag []*Tag []Tag []interface{}

```go
var tags []*Tag
...
// After reading tags
...
num, err := m2m.Add(tags)
if err == nil {
	fmt.Println("Added nums: ", num)
}
// It can pass multiple params
// m2m.Add(tag1, tag2, tag3)
```

### QueryM2Mer Remove

Remove tag from M2M relation:

Remove supports many types: Tag *Tag []*Tag []Tag []interface{}

```go
var tags []*Tag
...
// After reading tags
...
num, err := m2m.Remove(tags)
if err == nil {
	fmt.Println("Removed nums: ", num)
}
// It can pass multiple params
// m2m.Remove(tag1, tag2, tag3)
```

### QueryM2Mer Exist

Test if Tag is in M2M relation

```go
if m2m.Exist(&Tag{Id: 2}) {
	fmt.Println("Tag Exist")
}
```

### QueryM2Mer Clear

Clear all M2M relation

```go
nums, err := m2m.Clear()
if err == nil {
	fmt.Println("Removed Tag Nums: ", nums)
}
```

### QueryM2Mer Count

Count the number of Tags

```go
nums, err := m2m.Count()
if err == nil {
	fmt.Println("Total Nums: ", nums)
}
```

