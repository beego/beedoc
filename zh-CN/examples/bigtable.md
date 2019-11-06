---
name: 大表分表
sort: 4
---

# 大表取模分表

这个例子演示了如何使用 beego 对大表取模分表并使用orm增删改查:


```
package models

import (
	"fmt"
	"github.com/astaxie/beego"
	"github.com/astaxie/beego/orm"
)

type FileStat struct {
	Id int64 `orm:"auto;pk" json:"id"`
	FileId int64 `orm:"description(文件id)" json:"file_id"`
	Ctime int `orm:"index;description(创建时间)" json:"ctime"`
}

// 表名
func (u *FileStat) TableName() string {
	m := u.FileId % 100
	var table string
	if m < 10 {
		table = "file_stat_0" + orm.ToStr(m)
	} else {
		table = "file_stat_" + orm.ToStr(m)
	}
	return table
}

func init() {
	prefix := beego.AppConfig.String("mysql::prefix")
	orm.RegisterModelWithPrefix(prefix, new(FileStat))
}

func NewFileStat() *FileStat {
	return &FileStat{}
}

type FileStatList []*FileStat

// 操作数据库模型
type fileStatModel struct {
}

// 实例化
func NewFileStatModel() *fileStatModel {
	s := fileStatModel{}
	return &s
}

// 获取信息
func (this *fileStatModel) Get(fileId int64, id int64) (FileStat, error) {
	o := orm.NewOrm()
	n := FileStat{Id: id, FileId:fileId}
	err := o.Read(&n)
	if err != nil {
		return FileStat{}, err
	}
	return n, nil
}

// 更新信息
func (this *fileStatModel) Update(fileId int64, id int64, info map[string]interface{}) (int64, error) {
	stat := FileStat{FileId:fileId}
	o := orm.NewOrm()
	num, err := o.QueryTable(stat).Filter("id", id).Update(info)
	return num, err
}

// 插入信息
func (this *fileStatModel) Insert(fileId int64, n FileStat) (int64, error) {
	n.FileId = fileId
	o := orm.NewOrm()
	id, err := o.Insert(&n)
	return id, err
}

// 删除数据
func (this *fileStatModel) Delete(fileId int64, id int64) (int64, error) {
	stat := FileStat{Id: id, FileId:fileId}
	o := orm.NewOrm()
	return o.Delete(&stat);
}

//创建表
func (this *fileStatModel) CreateTable(fileId int64) error {
	o := orm.NewOrm()
	stat := FileStat{FileId: fileId}
	tableNew := beego.AppConfig.String("mysql::prefix") + stat.TableName()
	tableTpl := beego.AppConfig.String("mysql::prefix") + "file_stat_00"
	_, err := o.Raw(fmt.Sprintf("create table %s like %s", tableNew, tableTpl)).Exec()
	return err
}

//删除表
func (this *fileStatModel) DropTable(fileId int64) error {
	o := orm.NewOrm()
	stat := FileStat{FileId: fileId}
	tableDel := beego.AppConfig.String("mysql::prefix") + stat.TableName()
	_, err := o.Raw(fmt.Sprintf("drop table %s",tableDel)).Exec()
	return err
}
```
