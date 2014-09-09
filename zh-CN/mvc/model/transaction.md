---
name: 事务处理
sort: 7
---

# 事务处理

ORM 可以简单的进行事务操作

```go
o := NewOrm()
err := o.Begin()
// 事务处理过程
...
...
// 此过程中的所有使用 o Ormer 对象的查询都在事务处理范围内
if SomeError {
	err = o.Rollback()
} else {
	err = o.Commit()
}
```
