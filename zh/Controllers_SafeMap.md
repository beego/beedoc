# 线程安全的 Map

我们知道在 Go 语言里面 map 是非线程安全的，详细的 [atomic_maps](http://golang.org/doc/faq#atomic_maps)。但是我们在平常的业务中经常需要用到线程安全的 map，特别是在 goroutine 的情况下，所以 beego 内置了一个简单的线程安全的 map：

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

上面演示了如何使用线程安全的 Map，主要的接口有：

- Get(k interface{}) interface{}
- Set(k interface{}, v interface{}) bool
- Check(k interface{}) bool
- Delete(k interface{})
