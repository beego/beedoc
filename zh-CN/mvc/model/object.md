---
name: CRUD 操作
sort: 3
---

# 对象的CRUD操作

如果已知主键的值，那么可以使用这些方法进行CRUD操作

对 object 操作的四个方法 Read / Insert / Update / Delete

```go
o := orm.NewOrm()
user := new(User)
user.Name = "slene"

fmt.Println(o.Insert(user))

user.Name = "Your"
fmt.Println(o.Update(user))
fmt.Println(o.Read(user))
fmt.Println(o.Delete(user))
```

如果需要通过条件查询获取对象，请参见[高级查询](query.md#all)

## Read

```go
o := orm.NewOrm()
user := User{Id: 1}

err = o.Read(&user)

if err == orm.ErrNoRows {
	fmt.Println("查询不到")
} else if err == orm.ErrMissPK {
	fmt.Println("找不到主键")
} else {
	fmt.Println(user.Id, user.Name)
}
```

Read 默认通过查询主键赋值，可以使用指定的字段进行查询：

```go
user := User{Name: "slene"}
err = o.Read(&user, "Name")
...
```

对象的其他字段值将会是对应类型的默认值

复杂的单个对象查询参见 [One](query.md#one)

## Insert

```go
o := orm.NewOrm()
var user User
user.Name = "slene"
user.IsActive = true

fmt.Println(o.Insert(&user))
fmt.Println(user.Id)
```

创建后会自动对 auto 的 field 赋值

## Update

```go
o := orm.NewOrm()
user := User{Id: 1}
if o.Read(&user) == nil {
	user.Name = "MyName"
	o.Update(&user)
}
```

Update 默认更新所有的字段，可以更新指定的字段：

```go
// 只更新 Name
o.Update(&user, "Name")
// 指定多个字段
// o.Update(&user, "Field1", "Field2", ...)
...
```

根据复杂条件更新字段值参见 [Update](query.md#update)

## Delete

```go
o := orm.NewOrm()
o.Delete(&User{Id: 1})
```
Delete 操作会对反向关系进行操作，此例中 Post 拥有一个到 User 的外键。删除 User 的时候。如果 on_delete 设置为默认的级联操作，将删除对应的 Post

删除以后会清除 auto field 的值
