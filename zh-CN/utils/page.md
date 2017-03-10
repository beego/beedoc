---
name: 分页
sort: 2
---

# 分页

提供一个例子

结构

```go
type Page struct {
    PageNo     int
    PageSize   int
    TotalPage  int
    TotalCount int
    FirstPage  bool
    LastPage   bool
    List       interface{}
}
func PageUtil(count int, pageNo int, pageSize int, list interface{}) Page {
    tp := count / pageSize
    if count % pageSize > 0 {
        tp = count / pageSize + 1
    }
    return Page{PageNo: pageNo, PageSize: pageSize, TotalPage: tp, TotalCount: count, FirstPage: pageNo == 1, LastPage: pageNo == tp, List: list}
}
```

页面是使用 js 插件进行分页 https://github.com/lyonlai/bootstrap-paginator

```
<script type="text/javascript" src="/static/js/bootstrap-paginator.min.js"></script>
<script type="text/javascript">
  $(function () {
    $("#page").bootstrapPaginator({
      currentPage: '{{.Page.PageNo}}',
      totalPages: '{{.Page.TotalPage}}',
      bootstrapMajorVersion: 3,
      size: "small",
      onPageClicked: function(e,originalEvent,type,page){
        window.location.href = "/?p=" + page
      }
    });
  });
</script>
```
