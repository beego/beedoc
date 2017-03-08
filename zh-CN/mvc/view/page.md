---
name: 模板分页处理
sort: 4
---

# 分页处理
这里所说的分页，指的是大量数据显示时，每页显示固定的数量的数据，同时显示多个分页链接，用户点击翻页链接或页码时进入到对应的网页。
分页算法中需要处理的问题：
（1）当前数据一共有多少条。
（2）每页多少条，算出总页数。
（3）根据总页数情况，处理翻页链接。
（4）对页面上传入的 Get 或 Post 数据，需要从翻页链接中继续向后传。
（5）在页面显示时，根据每页数量和当前传入的页码，设置查询的 Limit 和 Skip，选择需要的数据。
（6）其他的操作，就是在 View 中显示翻页链接和数据列表的问题了。


模板处理过程中经常需要分页，那么如何进行有效的开发和操作呢？
我们开发组针对这个需求开发了如下的例子，希望对大家有用

- 工具类
https://github.com/beego/wetalk/blob/master/modules/utils/paginator.go
- 模板
https://github.com/beego/wetalk/blob/master/views/base/paginator.html
- 使用方法
https://github.com/beego/wetalk/blob/master/routers/base/base.go#L458
