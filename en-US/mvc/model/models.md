---
name: Model Definition
sort: 8
---

# Model Definition

Complicated model definition is not compulsory. It is　used for
database data converting and [Scheme Generating](cmd.md#自动建表)

Table name conversion :TODO is camel case to snake case.

	AuthUser -> auth_user
	Auth_User -> auth__user
	DB_AuthUser -> d_b__auth_user

Except the leading capital letter :TODO, replace capital case letter
into `_` and it's lower case. All the other `_` remain.

## Custom table name

```go
type User struct {
	Id int
	Name string
}

func (u *User) TableName() string {
	return "auth_user"
}
```

If you set [prefix](orm.md#registermodelwithprefix) to `prefix_`, the table name will be `prefix_auth_user`.

## Custom index

Add index to one or more fields:

```go
type User struct {
	Id    int
	Name  string
	Email string
}

// multiple fields index
func (u *User) TableIndex() [][]string {
	return [][]string{
		[]string{"Id", "Name"},
	}
}

// multiple fields unique key
func (u *User) TableUnique() [][]string {
	return [][]string{
		[]string{"Name", "Email"},
	}
}
```
## Custom engine

Only supports MySQL database

The default engine is the default engine of the current database engine of your mysql settings.

You can set `TableEngine` function in the model to choose the engine you want to use.

```go
type User struct {
	Id    int
	Name  string
	Email string
}

// Set engine to INNODB
func (u *User) TableEngine() string {
	return "INNODB"
}
```

## Set parameters

```go
orm:"null;rel(fk)"
```

Use `;` as the separator of multiple settings. Use `,` as the separator if a setting has multiple values.

#### Ignore field

Use `-` to ignore field in the struct.

```go
type User struct {
...
	AnyField string `orm:"-"`
...
}
```

#### auto

When Field type is int, int32, int64, uint, uint32 or uint64, you can set it as auto increment.

* If there is no primary key in the model definition, the field `Id` with one of the types above will be considered as auto increment key

Because of the design of go, even if you are using uint64, you can't use it's maximum. It is still treated as int64.

See issue [6113](http://code.google.com/p/go/issues/detail?id=6113)

#### pk

Set as primary key. Used for using other type field as primary key.

#### null

Fields are `NOT NULL` by default. Set null to `ALLOW NULL`.

```go
Name string `orm:"null"`
```

#### index

Add index for one field

#### unique

Add unique key for one field

```go
Name string `orm:"unique"`
```

#### column

Set column name in db table for field.

```go
Name string `orm:"column(user_name)"`
```

#### size

Default value for string field is varchar(255).

It will use varchar(size) after setting.

```go
Title string `orm:"size(60)"`
```

#### digits / decimals

Set precision for float32 or float64.

```go
Money float64 `orm:"digits(12);decimals(4)"`
```

Total 12 digits, 4 digits after point. For example: `12345678.1234`

#### auto_now / auto_now_add

```go
Created time.Time `orm:"auto_now_add;type(datetime)"`
Updated time.Time `orm:"auto_now;type(datetime)"`
```

* auto_now: every save will update time.
* auto_now_add: set time at the first save

This setting won't affect massive `update`.

#### type

If set type as date, the field's db type is date.

```go
Created time.Time `orm:"auto_now_add;type(date)"`
```

If set type as datetime, the field's db type is datetime.

```go
Created time.Time `orm:"auto_now_add;type(datetime)"`
```

#### default

Set default value for field with the same type. (Only support default value of cascade deleting. :TODO

```go
type User struct {
	...
	Status int `orm:"default(1)"`
	...
}
```

## Relationship

#### rel / reverse

**RelOneToOne**:

```go
type User struct {
	...
	Profile *Profile `orm:"null;rel(one);on_delete(set_null)"`
	...
}
```

The reverse relationship **RelReverseOne**:

```go
type Profile struct {
	...
	User *User `orm:"reverse(one)"`
	...
}
```

**RelForeignKey**:

```go
type Post struct {
	...
	User *User `orm:"rel(fk)"` // RelForeignKey relation
	...
}
```

The reverse relationship **RelReverseMany**:

```go
type User struct {
	...
	Posts []*Post `orm:"reverse(many)"` // reverse relationship of fk
	...
}
```

**RelManyToMany**:

```go
type Post struct {
	...
	Tags []*Tag `orm:"rel(m2m)"` // ManyToMany relation
	...
}
```

The reverse relationship **RelReverseMany**:

```go
type Tag struct {
	...
	Posts []*Post `orm:"reverse(many)"`
	...
}
```

#### rel_table / rel_through

This setting is for `orm:"rel(m2m)"` field

	rel_table       Set the auto generated m2m connecting table name
	rel_through     If you want to use custom m2m connecting table, set name by using this setting.
                  Format: pkg.path.byModelName
                  For example: app.models.PostTagRel PostTagRel table needs to have a relationship to Post table and Tag table.
                  

If rel_table is set, rel_through is ignored.

You can set these as follows: 

`orm:"rel(m2m);rel_table(the_table_name)"`

`orm:"rel(m2m);rel_through(pkg.path.ModelName)"`

#### on_delete

Set how to deal with field if related relationship is deleted:

	cascade        cascade delete (default)
	set_null       set to NULL. Need to set null = true
	set_default    set to default value. Need to set default value.
	do_nothing     do nothing. ignore.

```go
type User struct {
	...
	Profile *Profile `orm:"null;rel(one);on_delete(set_null)"`
	...
}
type Profile struct {
	...
	User *User `orm:"reverse(one)"`
	...
}

// Set User.Profile to NULL while deleting Profile
```

#### Examples of on_delete

```go
type User struct {
    Id int
    Name string
}

type Post struct {
    Id int
    Title string
    User *User `orm:"rel(fk)"`
}
```

Assume Post -> User is ManyToOne relationship by foreign key.

    o.Filter("Id", 1).Delete()

This will delete User with Id 1 and all his Posts.

If you don't want to delete the Posts, you need to set `set_null`

```go
type Post struct {
    Id int
    Title string
    User *User `orm:"rel(fk);null;on_delete(set_null)"`
}
```

In this case, only set related Post.user_id to NULL while deleting.

Usually for performance purposes, it doesn't matter to have redundant data. The massive deletion is the real problem

```go
type Post struct {
    Id int
    Title string
    User *User `orm:"rel(fk);null;on_delete(do_nothing)"`
}
```

So just don't change Post (ignore it) while deleting User.

## Model fields mapping with database type

Here is the recommended database type mapping. It's also the standard for table generation.

All the fields are **NOT NULL** by default.

#### MySQL

| go		   |mysql
| :---   	   | :---
| int, int32 - set as auto or name is `Id` | integer AUTO_INCREMENT
| int64 - set as auto or name is`Id` | bigint AUTO_INCREMENT
| uint, uint32 - set as auto or name is `Id` | integer unsigned AUTO_INCREMENT
| uint64 - set as auto or name is `Id` | bigint unsigned AUTO_INCREMENT
| bool | bool
| string - default size 255 | varchar(size)
| string - set type(text) | longtext
| time.Time - set type as date | date
| time.Time | datetime
| byte | tinyint unsigned
| rune | integer
| int | integer
| int8 | tinyint
| int16 | smallint
| int32 | integer
| int64 | bigint
| uint | integer unsigned
| uint8 | tinyint unsigned
| uint16 | smallint unsigned
| uint32 | integer unsigned
| uint64 | bigint unsigned
| float32 | double precision
| float64 | double precision
| float64 - set digits and decimals  | numeric(digits, decimals)

#### Sqlite3

| go		   | sqlite3
| :---   	   | :---
| int, int32, int64, uint, uint32, uint64 - set as auto or name is `Id` | integer AUTOINCREMENT
| bool | bool
| string - default size 255 | varchar(size)
| string - set type(text) | text
| time.Time - set type as date | date
| time.Time | datetime
| byte | tinyint unsigned
| rune | integer
| int | integer
| int8 | tinyint
| int16 | smallint
| int32 | integer
| int64 | bigint
| uint | integer unsigned
| uint8 | tinyint unsigned
| uint16 | smallint unsigned
| uint32 | integer unsigned
| uint64 | bigint unsigned
| float32 | real
| float64 | real
| float64 - set digits and decimals | decimal

#### PostgreSQL

| go		   | postgres
| :---   	   | :---
| int, int32, int64, uint, uint32, uint64 - set as auto or name is `Id` | serial
| bool | bool
| string - default size 255 | varchar(size)
| string - set type(text) | text
| time.Time - set type as date | date
| time.Time | timestamp with time zone
| byte | smallint CHECK("column" >= 0 AND "column" <= 255)
| rune | integer
| int | integer
| int8 | smallint CHECK("column" >= -127 AND "column" <= 128)
| int16 | smallint
| int32 | integer
| int64 | bigint
| uint | bigint CHECK("column" >= 0)
| uint8 | smallint CHECK("column" >= 0 AND "column" <= 255)
| uint16 | integer CHECK("column" >= 0)
| uint32 | bigint CHECK("column" >= 0)
| uint64 | bigint CHECK("column" >= 0)
| float32 | double precision
| float64 | double precision
| float64 - set digits and decimals  | numeric(digits, decimals)


## Relational fields

It's field type depends on related primary key.

* RelForeignKey
* RelOneToOne
* RelManyToMany
* RelReverseOne
* RelReverseMany
