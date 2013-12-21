---
name: Static files
sort: 6
---

# Handling static files

We talked about how to render a view. Usually there are lots of static files including images, js, css and so on. Our Beego project has already include these folders.

```
├── static
	│   ├── css
	│   ├── img
	│   └── js
```

Beego registers static directory as static path. Registering rule: URL prefix with directory mapping

	StaticDir["/static"] = "static"
	
You can register multiple static directories. For example there are two download directories `download1` and `download2` then you can set as:

	beego.SetStaticPath("/down1", "download1")	
	beego.SetStaticPath("/down2", "download2")	
	
Visting URL `http://localhost/down1/123.txt` will request `123.txt` under `download1` directory.
