---
name: 构造查询
sort: 6
---

# 构造查询

**QueryBuilder** 提供了一个简便，流畅的 SQL 查询构造器。在不影响代码可读性的前提下用来快速的建立 SQL 语句。

**QueryBuilder** 在功能上与 ORM 重合， 但是各有利弊。ORM 更适用于简单的 CRUD 操作，而 QueryBuilder 则更适用于复杂的查询，例如查询中包含子查询和多重联结。

使用方法:

```go
// User 包装了下面的查询结果
type User struct {
	Name string
	Age  int
}
var users []User

// 获取 QueryBuilder 对象. 需要指定数据库驱动参数。
// 第二个返回值是错误对象，在这里略过
qb, _ := orm.NewQueryBuilder("mysql")

// 构建查询对象
qb.Select("user.name",
	"profile.age").
	From("user").
	InnerJoin("profile").On("user.id_user = profile.fk_user").
	Where("age > ?").
	OrderBy("name").Desc().
	Limit(10).Offset(0)

// 导出 SQL 语句
sql := qb.String()

// 执行 SQL 语句
o := orm.NewOrm()
o.Raw(sql, 20).QueryRows(&users)
```

完整 API 接口:

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
