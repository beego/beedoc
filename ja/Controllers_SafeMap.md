# スレッドセーフマップ

Goではマップはスレッドセーフではありません、それをご存知でない場合は、この記事は役立つかもしれません: [atomic_maps](http://golang.org/doc/faq#atomic_map)。しかし、特にゴルーチンを使用している場合は、実際にはスレッドセーフマップのようなものを必要としています。それゆえ、beegoはシンプルなビルドインのスレッドセーフマップの実装を提供しています。

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

このマップは、次のインターフェイスがあります:

- Get(k interface{}) interface{}
- Set(k interface{}, v interface{}) bool
- Check(k interface{}) bool
- Delete(k interface{})
