---
name: 命令模式
sort: 9
---

# 命令模式

注册模型与数据库以后，调用 RunCommand 执行 orm 命令。

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
# 直接执行可以显示帮助
# 如果你的程序可以支持的话，直接运行 go run main.go orm 也是一样的效果
```

## 自动建表

```bash
./main orm syncdb -h
Usage of orm command: syncdb:
  -db="default": DataBase alias name
  -force=false: drop tables before create
  -v=false: verbose info
```

使用 `-force=1` 可以 drop table 后再建表

使用 `-v` 可以查看执行的 sql 语句

---

在程序中直接调用自动建表：

```go
// 数据库别名
name := "default"

// drop table 后再建表
force := true

// 打印执行过程
verbose := true

// 遇到错误立即返回
err := orm.RunSyncdb(name, force, verbose)
if err != nil {
	fmt.Println(err)
}
```

自动建表功能在非 force 模式下，是会自动创建新增加的字段的。也会创建新增加的索引。

对于改动过的旧字段，旧索引，需要用户自行进行处理。

## 打印建表SQL

```bash
./main orm sqlall -h
Usage of orm command: syncdb:
  -db="default": DataBase alias name
```

默认使用别名为 default 的数据库。
