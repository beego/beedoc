---
name: Test Cases
sort: 11
---

# ORM Test

Testing code:

* Model definition [models_test.go](https://github.com/beego/beego/blob/develop/client/orm/models_test.go)
* Test cases [orm_test.go](https://github.com/beego/beego/blob/develop/client/orm/orm_test.go)

#### MySQL

```bash
mysql -u root -e 'create database orm_test;'
export ORM_DRIVER=mysql
export ORM_SOURCE="root:@/orm_test?charset=utf8"
go test -v github.com/beego/beego/v2/core/client/orm
```


#### Sqlite3

```bash
touch /path/to/orm_test.db
export ORM_DRIVER=sqlite3
export ORM_SOURCE=/path/to/orm_test.db
go test -v github.com/beego/beego/v2/core/client/orm
```


#### PostgreSQL

```bash
psql -c 'create database orm_test;' -U postgres
export ORM_DRIVER=postgres
export ORM_SOURCE="user=postgres dbname=orm_test sslmode=disable"
go test -v github.com/beego/beego/v2/core/client/orm
```
