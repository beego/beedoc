---
name: CRUD Operations
sort: 3
---

# CRUD of Object

If already know the value of primary key, `Read`, `Insert`, `Update`, `Delete` can be used to manipulate the object.

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

To query the object by conditions see [Query in advance](query.md#all)

## Read

```go
o := orm.NewOrm()
user := User{Id: 1}

err = o.Read(&user)

if err == orm.ErrNoRows {
	fmt.Println("No result found.")
} else if err == orm.ErrMissPK {
	fmt.Println("No primary key found.")
} else {
	fmt.Println(user.Id, user.Name)
}
```

Read use primary key by default. Can use other fields as well:

```go
user := User{Name: "slene"}
err = o.Read(&user, "Name")
...
```
Other fields of the object are the default value according to the field type.

For detailed single object query, see [One](query.md#one)

## ReadOrCreate

Try to read a row from the database, or insert one if it doesn't exist.

Must input one condition field, also can input more field name.

```go
o := orm.NewOrm()
user := User{Name: "slene"}
// Three return values：Is Created，Object Id，Error
if created, id, err := o.ReadOrCreate(&user, "Name"); err != nil {
	if created {
		fmt.Println("New Insert an object. Id:", id)
	} else {
		fmt.Println("Get an object. Id:", id)
	}
}
```

## Insert

The first return value is auto inc Id value.

```go
o := orm.NewOrm()
var user User
user.Name = "slene"
user.IsActive = true

id, err := o.Insert(&user)
if err != nil {
	fmt.Println(id)
}
```

After creating it will assign values for auto fields.

## InsertMulti

Insert multi object in one api.

Like sql statement:

```
insert into table (name, age) values("slene", 28),("astaxie", 30),("unknown", 20)
```

The 1st param bulk is multi insert num in one time. The 2nd param is models slice.

The return value is success inserted num.

```go
users := []User{
	{Name: "slene"},
	{Name: "astaxie"},
	{Name: "unknown"},
	...
}
successNums, err := o.InsertMulti(100, users)
```

When bulk is equal 1, then insert model one by one.

## Update

The first return value is affected rows num.

```go
o := orm.NewOrm()
user := User{Id: 1}
if o.Read(&user) == nil {
	user.Name = "MyName"
	if num, err := o.Update(&user); err != nil {
		fmt.Println(num)
	}
}
```

Update updates all fields by default. You can update specificed fields:

```go
// Only update Name
o.Update(&user, "Name")
// Update multiple fields
// o.Update(&user, "Field1", "Field2", ...)
...
```

For detailed object update, see [One](query.md#one)

## Delete

The first return value is affected rows num.

```go
o := orm.NewOrm()
if num, err := o.Delete(&User{Id: 1}); err != nil {
	fmt.Println(num)
}
```

Delete will also manipulate reverse relationship. E.g.: `Post` has a foreign key to `User`. If on_delete is set to `cascade`, `Post` will be deleted while delete `User`.

After deleting it will clean up values for auto fields.

**Changed in 1.0.3** After deleting it will **not** clean up values for auto fields.
