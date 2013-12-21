---
name: View
sort: 5
---

# Creating views

When we create Controller we have something like `this.TplNames = "index.tpl"`. The view file support `tpl` and `html` extensions by default. You can call `beego.AddTemplateExt` to set other extensions. So how can views show the data we need? Beego is using the default template engine of Go so it's same as the Go templates. You can learn how to use Go template from [*Build Web Application with Golang*](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/07.4.md).

Let see a sample code:

```
<!DOCTYPE html>

<html>
  	<head>
    	<title>Beego</title>
    	<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
	</head>
  	
  	<body>
  		<header class="hero-unit" style="background-color:#A9F16C">
			<div class="container">
			<div class="row">
			  <div class="hero-text">
			    <h1>Welcome to Beego!</h1>
			    <p class="description">
			    	Beego is a simple & powerful Go web framework which is inspired by tornado and sinatra.
			    <br />
			    	Official website: <a href="http://{{.Website}}">{{.Website}}</a>
			    <br />
			    	Contact me: {{.Email}}
			    </p>
			  </div>
			</div>
			</div>
		</header>
	</body>
</html>
```

We assigned data into a map type variable `data` in Controller. Then we can get and output the data by key `.Website` and `Email`. 

Next let's talk about how to output static files.
