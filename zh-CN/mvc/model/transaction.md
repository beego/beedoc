---
name: 事务处理
sort: 7
---

ORM 可以简单的进行事务操作

ORM 操作事务，支持两种范式。一种通过闭包的方式，由 Beego 本身来管理事务的生命周期。
```go
	// Beego will manage the transaction's lifecycle
	// if the @param task return error, the transaction will be rollback
	// or the transaction will be committed
	err := o.DoTx(func(ctx context.Context, txOrm orm.TxOrmer) error {
		// data
		user := new(User)
		user.Name = "test_transaction"

		// insert data
		// Using txOrm to execute SQL
		_, e := txOrm.Insert(user)
		// if e != nil the transaction will be rollback
		// or it will be committed
		return e
	})
```

在这种方式里面，第一个参数是`task`，即该事务所有完成的动作。注意的是，如果它返回了 error，那么 Beego 会将整个事务回滚。

否则提交事务。

另外一个要注意的是，如果在`task`执行过程中，发生了`panic`，那么意味着 Beego 既不会帮你把事务回滚，也不会把事务提交。

我们推荐使用这种方式。

另外一种方式，则是传统的由开发自己手动管理事务的生命周期

```go
	o := orm.NewOrm()
	to, err := o.Begin()
	if err != nil {
		logs.Error("start the transaction failed")
		return
	}

	user := new(User)
	user.Name = "test_transaction"

	// do something with to. to is an instance of TxOrm

	// insert data
	// Using txOrm to execute SQL
	_, err = to.Insert(user)

	if err != nil {
		logs.Error("execute transaction's sql fail, rollback.", err)
		err = to.Rollback()
		if err != nil {
			logs.Error("roll back transaction failed", err)
		}
	} else {
		err = to.Commit()
		if err != nil {
			logs.Error("commit transaction failed.", err)
		}
	}
```

无论使用哪种方式，都应该注意到，只有通过`TxOrm`执行的 SQL 才会被认为是在一个事务里面。

```go
o := orm.NewOrm()
to, err := o.Begin()

// outside the txn
o.Insert(xxx)

// inside the txn
to.Insert(xxx)
```
