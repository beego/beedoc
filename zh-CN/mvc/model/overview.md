---
name: 概述
sort: 1
---

# 模型（Models）－ beego ORM

[![Build Status](https://drone.io/github.com/astaxie/beego/status.png)](https://drone.io/github.com/astaxie/beego/latest) [![Go Walker](http://gowalker.org/api/v1/badge)](http://gowalker.org/github.com/astaxie/beego/orm)

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

	go get github.com/astaxie/beego/orm

## 修改日志

* 2016-01-18: [规范了数据库驱动的命名](orm.md#registerdriver)
* 2014-03-10: [GetDB](orm.md#getdb) 从注册的数据库中返回 *sql.DB. [ResetModelCache](orm.md#resetmodelcache) 重置已注册的模型struct
* 2014-02-10: 随着beego1.1.0的发布提交的改进
  - 关于 [时区设置](orm.md#时区设置)

  - 新增的 api:
  Ormer.[InsertMulti](object.md#insertmulti)
  Ormer.[ReadOrCreate](object.md#readorcreate)
  RawSeter.[RowsToMap](rawsql.md#rowstomap)
  RawSeter.[RowsToStruct](rawsql.md#rowstostruct)
  orm.[NewOrmWithDB](orm.md#newormwithdb)

  - 改进的 api:
  RawSeter.[Values](rawsql.md#values) 支持设置 columns
  RawSeter.[ValuesList](rawsql.md#valueslist) 支持设置 columns
  RawSeter.[ValuesFlat](rawsql.md#valuesflat) 支持设置 column
  RawSeter.[QueryRow/QueryRows](rawsql.md#queryrow) 从对应每个strcut field位置的赋值，改为对应名称取值（不需要对应好字段数量与位置）

* 2013-10-14: [自动载入关系字段](query.md#载入关系字段)，[多对多关系操作](query.md#多对多关系操作)，完善[关系查询](query.md#关系查询)
* 2013-10-09: [原子操作更新值](query.md#update)
* 2013-09-22: [RegisterDataBase](orm.md#registerdatabase) maxIdle / maxConn 设置为可选参数, MySQL [自定义引擎](models.md#自定义引擎)
* 2013-09-16: 支持设置 空闲链接数 和 最大链接数 [SetMaxIdleConns](orm.md#setmaxidleconns) / [SetMaxOpenConns](orm.md#SetMaxOpenConns)
* 2013-09-12: [Read](object.md#read) 支持设定条件字段 [Update](object.md#update) / [All](query.md#all) / [One](query.md#one) 支持设定返回字段
* 2013-09-09: Raw SQL [QueryRow/QueryRows](rawsql.md#queryrow) 功能完成
* 2013-08-27: [自动建表](cmd.md#自动建表)继续改进
* 2013-08-19: [自动建表](cmd.md#自动建表)功能完成
* 2013-08-13: 更新数据库类型测试
* 2013-08-13: 增加 Go 类型支持，包括 int8、uint8、byte、rune 等
* 2013-08-13: 增强 date／datetime 的时区支持

## 快速入门

### 简单示例

```go
package main

import (
	"fmt"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql" // import your used driver
)

// Model Struct
type User struct {
	Id   int
	Name string `orm:"size(100)"`
}

func init() {
	// set default database
	orm.RegisterDataBase("default", "mysql", "root:root@/my_db?charset=utf8", 30)
	
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

## 文档索引

1. [Orm 使用方法](orm.md)
	- [数据库的设置](orm.md#数据库的设置)
		* [驱动类型设置](orm.md#registerdriver)
		* [参数设置](orm.md#registerdatabase)
		* [时区设置](orm.md#时区设置)
	- [注册模型](orm.md#注册模型)
	- [ORM 接口使用](orm.md#orm-接口使用)
	- [调试模式打印查询语句](orm.md#调试模式打印查询语句)
2. [对象的CRUD操作](object.md)
3. [高级查询](query.md)
	- [使用的表达式语法](query.md#expr)
	- [支持的操作符号](query.md#operators)
	- [高级查询接口使用](query.md#高级查询接口使用)
	- [关系查询](query.md#关系查询)
	- [载入关系字段](query.md#载入关系字段)
	- [多对多关系操作](query.md#多对多关系操作)
4. [使用SQL语句进行查询](rawsql.md)
5. [事务处理](transaction.md)
6. [模型定义](models.md)
	- [自定义表名](models.md#自定义表名)
	- [自定义引擎](models.md#自定义引擎)
	- [设置参数](models.md#设置参数)
	- [表关系设置](models.md#表关系设置)
	- [模型字段与数据库类型的对应](models.md#模型字段与数据库类型的对应)
7. [命令模式](cmd.md)
	- [自动建表](cmd.md#自动建表)
	- [打印建表SQL](cmd.md#打印建表sql)
8. [Test ORM](test.md)
9. [自定义字段](custom_fields.md)
10. [FAQ](faq.md)
