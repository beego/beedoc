---
name: Представление
sort: 5
---

# Создание шаблонов

Когда мы создали контроллер, мы использовали строку `this.TplNames = "index.tpl"`, чтобы объявить шаблон для отображения. По умолчанию `beego.Controller` поддерживает расширения `tpl` и `html`. Вы можете вызвать `beego.AddTemplateExt`, чтобы добавить другие расширения. Итак, как шаблоны могут отобразить необходимые вам данные? BeeGo использует стандартный механизм шаблонов, встроенный в Go, поэтому это шаблоны Go, простые и ясные. Вы можете изучите использование шаблонов Go тут: [*Build Web Application with Golang*](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/07.4.md).

Давайте взгянем на пример:

```
<!DOCTYPE html>

<html>
  	<head>
    	<title>BeeGo</title>
    	<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
	</head>
  	
  	<body>
  		<header class="hero-unit" style="background-color:#A9F16C">
			<div class="container">
			<div class="row">
			  <div class="hero-text">
			    <h1>Welcome to BeeGo!</h1>
			    <p class="description">
			    	BeeGo is a simple & powerful Go web framework which is inspired by tornado and sinatra.
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

Мы присвоили данные словарю `Data` в контроллере, который используется при отображении. Поэтому в настоящее время мы можем получить доступ к данным и их вывод по ключам `.Website` и `.Email`. 

Далее поговорим об использовании статических файлов.
