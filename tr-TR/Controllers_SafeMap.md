# Thread-safe map

We know that map is not thread safe in Go, if you don't know it, this article may be helpful for you: [atomic_maps](http://golang.org/doc/faq#atomic_maps). However, we need a kind of thread safe map in practice, especially when we are using goroutines. Therefore, Beego provides a simple built-in thread safe map implementation.

```go
bm := NewBeeMap()
if !bm.Set("astaxie", 1) {
	t.Error("set Error")
}
if !bm.Check("astaxie") {
	t.Error("check err")
}

if v := bm.Get("astaxie"); v.(int) != 1 {
	t.Error("get err")
}

bm.Delete("astaxie")
if bm.Check("astaxie") {
	t.Error("delete err")
}
```

This map has following interfaces:

- Get(k interface{}) interface{}
- Set(k interface{}, v interface{}) bool
- Check(k interface{}) bool
- Delete(k interface{})
