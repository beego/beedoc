---
name: Transaction
sort: 7
---

# Transaction

ORM has basic transaction support

```go
o := NewOrm()
err := o.Begin()
// transaction process
...
...
// All the query which are using o Ormer in this process are in the transaction
if SomeError {
	err = o.Rollback()
} else {
	err = o.Commit()
}
```
