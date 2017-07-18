---
name: CRUD 操作
sort: 3
---

# 对象的 CRUD 操作

如果已知主键的值，那么可以使用这些方法进行 CRUD 操作

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

err := o.Read(&user)

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

## ReadOrCreate

尝试从数据库读取，不存在的话就创建一个

默认必须传入一个参数作为条件字段，同时也支持多个参数多个条件字段

```go
o := orm.NewOrm()
user := User{Name: "slene"}
// 三个返回参数依次为：是否新创建的，对象 Id 值，错误
if created, id, err := o.ReadOrCreate(&user, "Name"); err == nil {
	if created {
		fmt.Println("New Insert an object. Id:", id)
	} else {
		fmt.Println("Get an object. Id:", id)
	}
}
```

## Insert

第一个返回值为自增健 Id 的值

```go
o := orm.NewOrm()
var user User
user.Name = "slene"
user.IsActive = true

id, err := o.Insert(&user)
if err == nil {
	fmt.Println(id)
}
```

创建后会自动对 auto 的 field 赋值

## InsertMulti

同时插入多个对象

类似sql语句

```
insert into table (name, age) values("slene", 28),("astaxie", 30),("unknown", 20)
```

第一个参数 bulk 为并列插入的数量，第二个为对象的slice

返回值为成功插入的数量

```go
users := []User{
	{Name: "slene"},
	{Name: "astaxie"},
	{Name: "unknown"},
	...
}
successNums, err := o.InsertMulti(100, users)
```

bulk 为 1 时，将会顺序插入 slice 中的数据

## Update

第一个返回值为影响的行数

```go
o := orm.NewOrm()
user := User{Id: 1}
if o.Read(&user) == nil {
	user.Name = "MyName"
	if num, err := o.Update(&user); err == nil {
		fmt.Println(num)
	}
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

第一个返回值为影响的行数

```go
o := orm.NewOrm()
if num, err := o.Delete(&User{Id: 1}); err == nil {
	fmt.Println(num)
}
```

Delete 操作会对反向关系进行操作，此例中 Post 拥有一个到 User 的外键。删除 User 的时候。如果 on_delete 设置为默认的级联操作，将删除对应的 Post

**Changed in 1.0.3** 删除以后不会删除 auto field 的值
