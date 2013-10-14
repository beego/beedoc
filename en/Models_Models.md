# Models

You don't have to define complex model, this feature is mainly for database conversion and [auto-create tables](Models_Cmd#auto-create-table).

Default rule of table name is camel name:

	AuthUser -> auth_user
	Auth_User -> auth__user
	DB_AuthUser -> d_b__auth_user

All upper case letters will be converted to lower case and prefix `_`, except the first letter.

## Customize table name

```go
type User struct {
	Id int
	Name string
}

func (u *User) TableName() string {
	return "auth_user"
}
```

If [prefix setting](Models_ORM#registermodelwithprefix) is `prefix_`, then the table name will be `prefix_auth_user`.

## Customize index

Add index(es) for one or more fields:

```go
type User struct {
	Id    int
	Name  string
	Email string
}

// Multiple fields index. 
func (u *User) TableIndex() [][]string {
	return [][]string{
		[]string{"Id", "Name"},
	}
}

// Multiple fields unique.
func (u *User) TableUnique() [][]string {
	return [][]string{
		[]string{"Name", "Email"},
	}
}
```

## Customize engine

Only support MySQL.

ORM uses default engine according to database default setting.

You can add `TableEngine` method to your struct to indicate engine:

```go
type User struct {
	Id    int
	Name  string
	Email string
}

// Set engine to be INNODB
func (u *User) TableEngine() string {
	return "INNODB"
}
```

## Struct Tag 

```go
orm:"null;rel(fk)"
```

Every field of struct tag contains 2 kinds of setting types. Multiple settings use `;` to separate, or use `,` to separate multiple values.

### Omit fields

Use `-` to omit fields of struct:

```go
type User struct {
...
	AnyField string `orm:"-"`
...
```

### auto

Set as Auto-increment Primary Key.

### pk

Set as Primary Key。

### null

Database tables use `NOT NULL` as default, set them to null means `ALLOW NULL`.

### blank

Allow string type fields can be empty, otherwise error occurs when clean operation.

### index

Add index for fields.

### unique

Add unique key for fields.

### column

Set database column name for fields:

```go
Name `orm:"column(user_name)"`
```

### size

Default type of string is `varchar(255)`.

After setting size of string type fields, the type of fields will be varchar(size):

```go
Title string `orm:"size(60)"`
```

### digits / decimals

Setting length and decimal of float32, float64:

```go
Money float64 `orm:"digits(12);decimals(4)"`
```

Total length is 12 and 4 decimals: `99999999.9999`.

### auto_now / auto_now_add

```go
Created     time.Time `auto_now_add`
Updated     time.Time `auto_now`
```

* auto_now: auto-update time when saved.
* auto_now_add: update time once when created.

This setting will not work when you do batch update.

### type

Type `date` for `time.Time` will be mapping to `date` type in database:

```go
Created time.Time `orm:"auto_now_add;type(date)"`
```

Type `text` for `string` will be mapping to `text` type in database:

```go
Content string `orm:"type(text)"`
```

### default

Set default value of fields with same type:

```go
type User struct {
	...
	Status int `orm:"default(1)"`
```

## Table relation

### rel / reverse

**RelOneToOne**:

```go
type User struct {
	...
	Profile *Profi	le `orm:"null;rel(one);on_delete(set_null)"`
```

Corresponding reverse relation **RelReverseOne**:

```go
type Profile struct {
	...
	User *User `orm:"reverse(one)"`
```

**RelForeignKey**:

```go
type Post struct {
	...
	User*User `orm:"rel(fk)"` // RelForeignKey relation
```

Corresponding reverse relation **RelReverseMany**:

```go
type User struct {
	...
	Posts []*Post `orm:"reverse(many)"` // fk 的反向关系
```

**RelManyToMany**:

```go
type Post struct {
	...
	Tags []*Tag `orm:"rel(m2m)"` // ManyToMany relation
```

Corresponding reverse relation **RelReverseMany**:

```go
type Tag struct {
	...
	Posts []*Post `orm:"reverse(many)"`
```

### rel_table / rel_through

This setting is for relation fields that uses `orm:"rel(m2m)"`:

	rel_table       Auto-generate m2m relation table name
	rel_through     If you want to use customize m2m relation tables in m2m 					relation, you have to set this with format pkg.path.ModelName
	                eg: app.models.PostTagRel
	                Table PostTagRel need have relation Post and Tag.

`rel_through` will be omitted after you setting `rel_table`.

Usage:

`orm:"rel(m2m);rel_table(the_table_name)"`

`orm:"rel(m2m);rel_through(pkg.path.ModelName)"`

### on_delete

This describes how to deal with relation fields when one of them have delete operation:

	cascade        default
	set_null       use `null = true` to set to NULL
	set_default    set as default, need to set default value
	do_nothing     

```go
type User struct {
	...
	Profile *Profile `orm:"null;rel(one);on_delete(set_null)"`
...
type Profile struct {
	...
	User *User `orm:"reverse(one)"`

// User.Profile will be setting to NULL after Profile was deleted.
```

#### on_delete example

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

Suppose Post -> User is ManyToOne relation, which is foreign key.

	o.Filter("Id", 1).Delete()

All posts that posted by User that Id is 1 will be deleted at the same time.

If you do not want to delete them, need to setting `set_null`:

```go
type Post struct {
    Id int
    Title string
    User *User `orm:"rel(fk);null;on_delete(set_null)"`
}
```

Now the posts will not be deleted but the Post.user_id will all be setting to NULL.

For performance purpose, you can do following to ignore delete operation:

```go
type Post struct {
    Id int
    Title string
    User *User `orm:"rel(fk);null;on_delete(do_nothing)"`
}
```

## Mapping of model type and database type

This standard will work for auto-create table.

All fields are **NOT NULL** as default.

### MySQL

| go		   |mysql
| :---   	   | :---
| bool | bool
| string - when setting size | varchar(size)
| string | longtext
| time.Time - when setting type to date | date
| time.TIme | datetime
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
| float64 - when setting digits and decimals | numeric(digits, decimals)

### Sqlite3

| go		   | sqlite3
| :---   	   | :---
| bool | bool
| string - when setting size | varchar(size)
| string | text
| time.Time - when setting type to date | date
| time.TIme | datetime
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
| float64 - when setting digits and decimals  | decimal

### PostgreSQL

| go		   | postgres
| :---   	   | :---
| bool | bool
| string - when setting size  | varchar(size)
| string | text
| time.Time - when setting type to date | date
| time.TIme | timestamp with time zone
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
| float64 - when setting digits and decimals  | numeric(digits, decimals)


## Relational fields

The field type is depending on its primary key:

* RelForeignKey
* RelOneToOne
* RelManyToMany
* RelReverseOne
* RelReverseMany