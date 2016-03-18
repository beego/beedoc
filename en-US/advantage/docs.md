---
name: Automated API Documentation
sort: 2
---

# Automated API Document

Automated documentation is a very cool feature that I found to be desirable. Now it became a reality in Beego. As I said Beego will not only boost the development of your API but also make the API easy to use for the user. 

Beego implemented the [swagger specification](http://swagger.io/) for API documentation. It's very easy to create powerful interactive API documentation.

Ok, let's try it out now. First let's create a new API application by `bee api beeapi`

# API global settings

Add the following comments at the top of `routers/router.go`:

```
// @APIVersion 1.0.0
// @Title mobile API
// @Description mobile has every tool to get any job done, so codename for the new mobile APIs.
// @Contact astaxie@gmail.com
package routers
```

The comments above set the global information. The available settings:

- @APIVersion
- @Title
- @Description
- @Contact
- @TermsOfServiceUrl
- @License
- @LicenseUrl

## Router Parsing
Right now automated API documentation only supports `NSNamespace` and `NSInclude` and it only supports two levels of parsing. The first level is the API version and the second level is the modules.

```
func init() {
	ns :=
		beego.NewNamespace("/v1",
			beego.NSNamespace("/customer",
				beego.NSInclude(
					&controllers.CustomerController{},
					&controllers.CustomerCookieCheckerController{},
				),
			),
			beego.NSNamespace("/catalog",
				beego.NSInclude(
					&controllers.CatalogController{},
				),
			),
			beego.NSNamespace("/newsletter",
				beego.NSInclude(
					&controllers.NewsLetterController{},
				),
			),
			beego.NSNamespace("/cms",
				beego.NSInclude(
					&controllers.CMSController{},
				),
			),
			beego.NSNamespace("/suggest",
				beego.NSInclude(
					&controllers.SearchController{},
				),
			),
		)
	beego.AddNamespace(ns)
}
```

## Application Comment
This is the most important part of comment. For example:

```
package controllers

import "github.com/astaxie/beego"

// CMS API
type CMSController struct {
	beego.Controller
}

func (c *CMSController) URLMapping() {
	c.Mapping("StaticBlock", c.StaticBlock)
	c.Mapping("Product", c.Product)
}

// @Title getStaticBlock
// @Description get all the staticblock by key
// @Param	key		path 	string	true		"The static block key."
// @Success 200 {object} ZDT.ZDTMisc.CmsResponse
// @Failure 400 Bad request
// @Failure 404 Not found
// @router /staticblock/:key [get]
func (c *CMSController) StaticBlock() {

}

// @Title Get Product list
// @Description Get Product list by some info
// @Success 200 {object} models.ZDTProduct.ProductList
// @Param	category_id		query	int	false		"category id"
// @Param	brand_id	query	int	false		"brand id"
// @Param	query	query	string 	false		"query of search"
// @Param	segment	query	string 	false		"segment"
// @Param	sort 	query	string 	false		"sort option"
// @Param	dir 	query	string 	false		"direction asc or desc"
// @Param	offset 	query	int		false		"offset"
// @Param	limit 	query	int		false		"count limit"
// @Param	price 			query	float		false		"price"
// @Param	special_price 	query	bool		false		"whether this is special price"
// @Param	size 			query	string		false		"size filter"
// @Param	color 			query	string		false		"color filter"
// @Param	format 			query	bool		false		"choose return format"
// @Failure 400 no enough input
// @Failure 500 get products common error
// @router /products [get]
func (c *CMSController) Product() {

}
```

In the code above, we defined the comment on top of `CMSController` is the information for this module. Then we defined the comment for every controller's methods. 

Below is a list of supported comments for generating swagger APIs:

- @Title

	The title for this API. It's a string, and all the content after the first space will be parsed as the title.

- @Description

	The description for this API. It's a string, and all the content after the first space will be parsed as the description.

- @Param

	`@Param` defines the parameters sent to the server. There are five columns for each `@Param`:
	1. parameter key;
	2. parameter sending type; It can be `form`, `query`, `path`, `body` or `header`. `form` means the parameter sends by POST. `query` means the parameter sends by GET in url. 
	`path` means the parameter in the url path, such as key in the former example. `body` means the raw data send from request body. `header` means the parameter is in request header.
	3. parameter data type
	4. required
	5. comment

- @Success

	The success message returned to client. Three parameters.
	1. status code.
	2. return type; Must wrap with {}.
	3. returned object or string. For {object}, use path and the object name of your project here and `bee` tool will look up the object while generating the docs. For example `models.ZDTProduct.ProductList` represents `ProductList` object under `/models/ZDTProduct`

	>>> Use space to separate these three parameters

- @Failure

	The failure message returned to client. Two parameters separated by space.
	1. Status code.
	2. Error message.

- @router

	Router information. Two parameters separated by space.
	1. The request's router address.
	2. Supported request methods. Wrap in `[]`. Use `,` to separate multiple methods.

## Generate document automatically

Make it work by following the steps:
1. Enable docs by setting `EnableDocs = true` in `conf/app.conf`.
3. Import `_ "beeapi/docs"` in `main.go`.
4. Use `bee run -downdoc=true -gendoc=true` to run your API application and rebuild document automatically.
5. Visit `swagger document from API project's URL and port.  (see item #1 below)

Your API document is available now. Open your browser and check it.

![](../images/docs.png)

![](../images/doc_test.png)

## Problems You May Have
1. CORS
	Two solutions
	1. Integrate `swagger` into the application. Download [swagger](https://github.com/beego/swagger/releases) and put it into project folder. (`bee run -downdoc=true` will also download it and put it into project folder)
	And before 	`beego.Run()` in `func main()` of `main.go`

		if beego.RunMode == "dev" {
			beego.DirectoryIndex = true
			beego.StaticDir["/swagger"] = "swagger"
		}

	And then visit `swagger` document from API project's URL and port.
	2. Make API support CORS

			ctx.Output.Header("Access-Control-Allow-Origin", "*")

2. Other problems. This is a feature used in my own project. If you have some other problems please fire issues to us.
