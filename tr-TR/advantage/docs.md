---
name: Automated API Documentation
sort: 2
---

# Otomatik API Dökümantasyonu

Otomatik dökümantasyon çok arzu edilen, çok havalı bir özellik. Bu özellik Beego'ya da eklendi. Beego sadece API development hızınızı arttırmayacak, ayrıca kolay bir API kullanımı sunacak.

Beego'ya API dökümantasyonu için [swagger](http://swagger.io/) entegre edilmiştir. Swagger çok kolay, çok güçlü API dökümantasyonu oluşturur.

Tamam, hadi şimdi deneyelim. Öncelikle aşağıdaki komutla yeni bir API uygulaması oluşturalım :  

`bee api beeapi`


# API global ayarları

Aşağıdaki yorum satırlarını `routers/router.go` dosyasının üstüne ekleyelim :


```
// @APIVersion 1.0.0
// @Title mobile API
// @Description mobile has every tool to get any job done, so codename for the new mobile APIs.
// @Contact astaxie@gmail.com
package routers
```

Yukarıdaki kod bloğunda gördüğünüz global tanımlamalar ve daha fazlası aşağıdaki gibidir :

- @APIVersion
- @Title
- @Description
- @Contact
- @TermsOfServiceUrl
- @License
- @LicenseUrl
- @Name
- @URL
- @LicenseUrl
- @License
- @Schemes
- @Host

## Router Eşleştirme

Şimdilik otomatik API dökümantasyonu  `NSNamespace`  ve  `NSInclude`  seviye tanımlamalarını destekliyor. İlk seviye API versiyonunu, ikinci seviye ise modülleri temsil eder.


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

## Uygulama Yorumları

Bu kısım yorum eklemenin en önemli kısmıdır. Örnek verirsek :

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
// @Summary getStaticBlock
// @Deprecated Deprecated
// @Description get all the staticblock by key
// @Param	key	path	string	true	"The static block key."	default_value
// @Success 200 {object} ZDT.ZDTMisc.CmsResponse
// @Failure 400 Bad request
// @Failure 404 Not found
// @Accept json
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

Yukarıdaki kod içerisinde `CMSController`'ın üzerine modülün açıklamasını içeren bir yorum ekledik. Sonrasında her controller metoduna yorumları tanımladık.

Aşağıdaki liste Swagger'ın desteklediği yorum tanımlamalarıdır :

- @Accept
	Kabul edilen tipleri ifade eder (json/xml/html/plain)
- @Deprecated
	Artık kullanılmamasını gereken yerleri ifade eder
- @Title

	API'ın başlığıdır. String tipindedir ve tanımlama ilk boşluktan sonra başlar.

- @Description

	API için açıklama kısmıdır. String tipindedir ve tanımlama ilk boşluktan sonra başlar.

- @Param

	`@Param` defines the parameters sent to the server. There are five columns for each `@Param`:
	1. parameter key;
	2. parameter sending type; It can be `formData`, `query`, `path`, `body` or `header`. `formData` means the parameter sends by POST ( set Content-Type to application/x-www-form-urlencoded ) . `query` means the parameter sends by GET in url. 
	`path` means the parameter in the url path, such as key in the former example. `body` means the raw data send from request body. `header` means the parameter is in request header.
	3. parameter data type
	4. required
	5. comment
	6. default value

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
2. Use `bee run -downdoc=true -gendoc=true` to run your API application and rebuild document automatically.
3. Visit `swagger document from API project's URL and port.  (see item #1 below)

Your API document is available now. Open your browser and check it.

![](../images/docs.png)

![](../images/doc_test.png)

## Problems You May Have
1. CORS
	Two solutions:
	1. Integrate `swagger` into the application. Download [swagger](https://github.com/beego/swagger/releases) and put it into project folder. (`bee run -downdoc=true` will also download it and put it into project folder)
	And before 	`beego.Run()` in `func main()` of `main.go`
		```go
		if beego.BConfig.RunMode == "dev" {
			beego.BConfig.WebConfig.DirectoryIndex = true
			beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"
		}
		```
		And then visit `swagger` document from API project's URL and port.

	2. Make API support CORS
		```go
		ctx.Output.Header("Access-Control-Allow-Origin", "*")
		```
2. Other problems. This is a feature used in my own project. If you have some other problems please fire issues to us.
