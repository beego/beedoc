---
name: Command Line
sort: 9
---

# Command Line

You can call `orm.RunCommand()` after you registered models and database(s) as follows:

```go
func main() {
	// orm.RegisterModel...
	// orm.RegisterDataBase...
	...
	orm.RunCommand()
}
```

```bash
go build main.go
./main orm
# Get help by just run it.
# If possible, go run main.go orm has the same result.
```

## Database Schema Generation

```bash
./main orm syncdb -h
Usage of orm command: syncdb:
  -db="default": DataBase alias
  -force=false: drop tables before create
  -v=false: verbose info
```

Use the `-force=1` flag to force drop tables and re-create.

Use the `-v` flag to print SQL statements.

---

Use program to create tables:

```go
// Database alias.
name := "default"

// Drop table and re-create.
force := true

// Print log.
verbose := true

// Error.
err := orm.RunSyncdb(name, force, verbose)
if err != nil {
	fmt.Println(err)
}
```

Even if you do not enable `force` mode, ORM also will auto-add new fields and indexes, but you have to deal with delete operations yourself.

## Print SQL Statements

```bash
./main orm sqlall -h
Usage of orm command: syncdb:
  -db="default": DataBase alias name
```

Use database with alias `default` as default.
