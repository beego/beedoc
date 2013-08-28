# CRUD operations of object

There are 3 simple methods to operate objects: Read/Insert/Update/Delete.

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

## Read

```go
o := orm.NewOrm()
user := User{Id: 1}

err = o.Read(&user)

if err == orm.ErrNoRows {
	fmt.Println("no result found")
} else if err == orm.ErrMissPK {
	fmt.Println("cannot find primary key")
} else {
	fmt.Println(user.Id, user.Name)
}
```

## Insert

```go
o := orm.NewOrm()
var user User
user.Name = "slene"
user.IsActive = true

fmt.Println(o.Insert(&user))
fmt.Println(user.Id)
```

Field `auto` will be assigned value automatically.

## Update

```go
o := orm.NewOrm()
user := User{Id: 1}
if o.Read(&user) == nil {
	user.Name = "MyName"
	o.Update(&user)
}
```

### Delete

```go
o := orm.NewOrm()
o.Delete(&User{Id: 1})
```

The `Delete` operation will do reverse relation operation, which in this case, `Post` has a outside key `User`; therefore, `Post` will be deleted when you deleted the `User` if you enabled `on_delete` option.

Value of `auto` will be clear as well.
