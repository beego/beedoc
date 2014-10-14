#  map seguro para hilos

Sabemos que map no es seguro para hilos (o subprocesos) en Go, si tú no lo sabes, este artículo pude ser de gran ayuda para ti: [atomic_maps](http://golang.org/doc/faq#atomic_maps). Sin embargo, necesitamos un tipo de map seguro para hilos en práctica, especialmente cuando estamos usando goroutines. Por lo tanto, beego provee una implementación simple built-in de map seguro para hilos.

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

Este map tiene las siguientes interfaces:

- Get(k interface{}) interface{}
- Set(k interface{}, v interface{}) bool
- Check(k interface{}) bool
- Delete(k interface{})
