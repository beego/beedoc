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

## Insert

```go
o := orm.NewOrm()
var user User
user.Name = "slene"
user.IsActive = true

fmt.Println(o.Insert(&user))
fmt.Println(user.Id)
```

After creating it will assign values for auto fields.

## Update

```go
o := orm.NewOrm()
user := User{Id: 1}
if o.Read(&user) == nil {
	user.Name = "MyName"
	o.Update(&user)
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

```go
o := orm.NewOrm()
o.Delete(&User{Id: 1})
```

Delete will also manipulate reverse relationship. E.g.: `Post` has a foreign key to `User`. If on_delete is set to `cascade`, `Post` will be deleted while delete `User`.

After deleting it will clean up values for auto fields.
