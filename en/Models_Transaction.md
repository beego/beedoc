# Transcation

ORM can do simple transcations:

```go
o := NewOrm()
err := o.Begin()
...
...
// All objects of o Ormer are in the range of transcation.
if SomeError {
	err = o.Rollback()
} else {
	err = o.Commit()
}
```
