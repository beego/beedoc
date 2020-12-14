---
name: 概述
sort: 1
---

# 模型（Models）－ beego ORM

beego ORM 是一个强大的 Go 语言 ORM 框架。她的灵感主要来自 Django ORM 和 SQLAlchemy。

目前该框架仍处于开发阶段，可能发生任何导致不兼容的改动。

**已支持数据库驱动：**

* MySQL：[github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
* PostgreSQL：[github.com/lib/pq](https://github.com/lib/pq)
* Sqlite3：[github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3)

以上数据库驱动均通过基本测试，但我们仍需要您的反馈。

**ORM 特性：**

* 支持 Go 的所有类型存储
* 轻松上手，采用简单的 CRUD 风格
* 自动 Join 关联表
* 跨数据库兼容查询
* 允许直接使用 SQL 查询／映射
* 严格完整的测试保证 ORM 的稳定与健壮

更多特性请在文档中自行品读。

**安装 ORM：**

	go get github.com/beego/beego/v2/client/orm

## 快速入门

### 简单示例

```go
package main

import (
	"fmt"
	"github.com/beego/beego/v2/client/orm"
	_ "github.com/go-sql-driver/mysql" // import your used driver
)

// Model Struct
type User struct {
	Id   int
	Name string `orm:"size(100)"`
}

func init() {
	// set default database
	orm.RegisterDataBase("default", "mysql", "username:password@tcp(127.0.0.1:3306)/db_name?charset=utf8&loc=Local", 30)

	// register model
	orm.RegisterModel(new(User))

	// create table
	orm.RunSyncdb("default", false, true)
}

func main() {
	o := orm.NewOrm()

	user := User{Name: "slene"}

	// insert
	id, err := o.Insert(&user)
	fmt.Printf("ID: %d, ERR: %v\n", id, err)

	// update
	user.Name = "astaxie"
	num, err := o.Update(&user)
	fmt.Printf("NUM: %d, ERR: %v\n", num, err)

	// read one
	u := User{Id: user.Id}
	err = o.Read(&u)
	fmt.Printf("ERR: %v\n", err)

	// delete
	num, err = o.Delete(&u)
	fmt.Printf("NUM: %d, ERR: %v\n", num, err)
}
```

### 关联查询

```go
type Post struct {
	Id    int    `orm:"auto"`
	Title string `orm:"size(100)"`
	User  *User  `orm:"rel(fk)"`
}

var posts []*Post
qs := o.QueryTable("post")
num, err := qs.Filter("User__Name", "slene").All(&posts)
```

### SQL 查询

当您无法使用 ORM 来达到您的需求时，也可以直接使用 SQL 来完成查询／映射操作。

```go
var maps []orm.Params
num, err := o.Raw("SELECT * FROM user").Values(&maps)
for _,term := range maps{
	fmt.Println(term["id"],":",term["name"])
}
```

### 事务处理

```go
o.Begin()
...
user := User{Name: "slene"}
id, err := o.Insert(&user)
if err == nil {
	o.Commit()
} else {
	o.Rollback()
}
```

### 调试查询日志

在开发环境下，您可以使用以下指令来开启查询调试模式：

```go
func main() {
	orm.Debug = true
...
```

开启后将会输出所有查询语句，包括执行、准备、事务等。

例如：

```go
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.4ms] - 	[INSERT INTO `user` (`name`) VALUES (?)] - `slene`
...
```

注意：我们不建议您在部署产品后这样做。
