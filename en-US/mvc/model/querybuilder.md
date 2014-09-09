---
name: Query Builder
sort: 6
---

# Query Builder

**QueryBuilder** provides an API for fluently construting SQL DML (Data Manipulation Language).
It provides a set of methods enables developers to easily construct SQL query without compromising readability.

It serves as an alternative to ORM, you may choose the one you prefer.
We recommend using ORM for simple CRUD operations, whereas QueryBuilder for complex queries with subqueries,
nested queries and deep joins.

Usage exmple:

```go
// User is a wrapper for result row in this example
type User struct {
	Name string
	Age  int
}
var users []User

// Get a QueryBuilder object. Takes DB driver name as parameter
// Second parameter is error, ignored here
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