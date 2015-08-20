---
name: Query Builder
sort: 6
---

# Query Builder

**QueryBuilder** provides an API for convenient and fluent construction of SQL queries. It consists of a set of methods enabling developers to easily construct SQL queries without compromising readability.

It serves as an alternative to ORM. ORM is more for simple CRUD operations, whereas QueryBuilder is for complex queries with subqueries and multi-joins.

Usage example:

```go
// User is a wrapper for result row in this example
type User struct {
	Name string
	Age  int
}
var users []User

// Get a QueryBuilder object. Takes DB driver name as parameter
// Second return value is error, ignored here
qb, _ := orm.NewQueryBuilder("mysql")

// Construct query object
qb.Select("user.name",
	"profile.age").
	From("user").
	InnerJoin("profile").On("user.id_user = profile.fk_user").
	Where("age > ?").
	OrderBy("name").Desc().
	Limit(10).Offset(0)

// export raw query string from QueryBuilder object
sql := qb.String()

// execute the raw query string
o := orm.NewOrm()
o.Raw(sql, 20).QueryRows(&users)
```

Full API interface:

```go
type QueryBuilder interface {
	Select(fields ...string) QueryBuilder
	From(tables ...string) QueryBuilder
	InnerJoin(table string) QueryBuilder
	LeftJoin(table string) QueryBuilder
	RightJoin(table string) QueryBuilder
	On(cond string) QueryBuilder
	Where(cond string) QueryBuilder
	And(cond string) QueryBuilder
	Or(cond string) QueryBuilder
	In(vals ...string) QueryBuilder
	OrderBy(fields ...string) QueryBuilder
	Asc() QueryBuilder
	Desc() QueryBuilder
	Limit(limit int) QueryBuilder
	Offset(offset int) QueryBuilder
	GroupBy(fields ...string) QueryBuilder
	Having(cond string) QueryBuilder
	Subquery(sub string, alias string) string
	String() string
}
```