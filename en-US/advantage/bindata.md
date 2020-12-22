---
name: Bindata
sort: 3
---
# Bindata

This chapter will go over how to use bindata to compile a single binary with all of your assets.
The goal is to have a single binary that does not rely on the underlying OS.
Which is great for cloud based projects that have an ephemeral filesystem.


# Preface

You will need the bindata libraries. Follow the installation instructions at https://github.com/elazarl/go-bindata-assetfs
```
go get github.com/jteeuwen/go-bindata/...
go get github.com/elazarl/go-bindata-assetfs/...
```
I like to keep my root directory clean and as such, have moved some common directories to sub-directories. For this example the directory structure will be as such.
```
├───app
│   ├───controllers
│   ├───models
│   ├───routers
│   └───views
│       ├───content
│       ├───includes
│       └───layouts
├───bindata
│   ├───conf
│   ├───static
│   └───views
├───conf
├───src
│   ├───css
│   ├───dart
│   ├───images
│   ├───js
│   └───vendor
├───static
│   ├───css
│   ├───img
│   └───js
└───tests
```
	
## Special Notes

Bindata Paths.
I'm not sure why, but when you reference data in the bindata file system, you need to omit the root of the path. I am not sure why it cuts the first directory off.
```
Ex.
If you want to access ./static/css/app.css
You will reference it as css/app.css
```

Beego file system interface.
In order to interact with the bindata file system we need a custom interface to call. In this setup, each bindata asset group is in its own package and the function we need is private. Therefore, since the bindata.go file will be overwritten each time we recompile our assets, we need a separate gofile to handle this interface.
In each asset folder (bindata/views, bindata/static, etc...) We need to create the following code in a new file.
```
// Make sure you package matches the bindata.go package
package staticAssets
  
import (  
	"net/http"  
	"github.com/astaxie/beego" "github.com/elazarl/go-bindata-assetfs"
 )  
  
// Create an empty file system struct
type FS struct {}  
  
// This function allows callers to open the virtual file
// This is where the magic happens.
func (d FS) Open(name string) (http.File, error) {  
   // beego.Debug("Looking for file: " + name)  
   asset := assetFS()  
   return asset.Open(name)  
}  
  
// This function exposes the assetFS() function in bindata.go
// This feature is currently not utilized,
// but could be helpful for future implimentations.
func AssetFS() *assetfs.AssetFS {  
   return assetFS()  
}
```

## View Templates

Compile your view templates, make sure that the bindata folder exits.
```
go-bindata-assetfs -ignore=\\.go -o "./bindata/views/bindata.go" -pkg "viewAssets" app/views/...
```

Now we need to tell Beego how to access these templates.
In your main.go add this to your init function. 
```
func init() {
	beego.SetTemplateFSFunc(func() http.FileSystem {  
	   return viewAssets.FS{}  
	})
}
```
Notice how we are calling viewAssets.FS{}
viewAssets is the package name of the bindata.go that holds our views.
FS{} is the empty struct with the Open interface.
Beego will recurse through all of the subdirectories in our compiled file system.


## Static Assets

Compile your static assets folder, make sure that the bindata folder exits.
```
go-bindata-assetfs -ignore=\\.go -o "./bindata/static/bindata.go" -pkg "staticAssets" ./static/...
```

Beego is going to attempt to handle the /static URL by default. We need to disable this feature. In your main.go add this to your init function.
```
func init() {
	beego.DelStaticPath("/static")
}
```
Now we need to handle requests for your static assets.
In your router, register the following handler.
```
beego.Handler("/static", http.StripPrefix("/static/", http.FileServer(staticAssets.FS{})), true)
```

## Conf files (WIP)

This is a work in progress. 
I utilize this to load a config.json that holds non-Beego configuration options.

Compile your configuration files, make sure that the bindata folder exits.
```
go-bindata-assetfs -ignore=\\.go -o "./bindata/conf/bindata.go" -pkg "confAssets" ./conf/...
```
For my use case, with the following code, I can detect if a config file exists locally or if I need to load from bindata. Notice with this function we do not strip the leading folder as in the other examples. We are using the Asset() function that is generated in bindata.go
```
const configFilePath =	"conf/config.json"
var Compiled =  		false

func Start() {
	var configData []byte  
	beego.Info("Reading Config...")  
	if FileExists(configFilePath) {  
	   beego.Info("File exists in filesystem")  
	   configData, err = ioutil.ReadFile(configFilePath)  
	   if err != nil {  
	      beego.Error(err)  
	   }  
	} else {  
	   beego.Info("Looking in bindata")  
	   configData, err = confAssets.Asset(configFilePath)  
	   if err != nil {  
	      beego.Error(err)  
	   }  
	   Compiled = true  
	}  
	beego.Info("Loading Config")  
	err = json.Unmarshal(configData, &Config)  
	if err != nil {  
	   beego.Error(err)  
	}
	
	// Program logic here...
}
```
