---
name: Pagination 
sort: 4
---

# Pagination

We use pagination in template very often. How can we do that?
We create these demo for pagination. Hope it's useful for you.

## Controllers

Before you can use the paginator in the view you have to set it in your controller:

    package controllers

    type PostsController struct {
      beego.Controller
    }
    
    func (this *PostsController) ListAllPosts() {
        // sets this.Data["paginator"] with the current offset (from the url query param)
        postsPerPage := 20
      	paginator := pagination.SetPaginator(this, postsPerPage, CountPosts())

        // fetch the next 20 posts
        this.Data["posts"] = ListPostsByOffsetAndLimit(paginator.Offset(), postsPerPage)
    }

## Views

Example templates (using Twitter Bootstrap):

https://github.com/beego/wetalk/blob/master/views/base/paginator.html

