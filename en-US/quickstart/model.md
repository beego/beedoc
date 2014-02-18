---
name: Model
sort: 4
---

# Creating models

We know we are using database a lot in web applications so model is usually handle these jobs. There is no demo for Model in our `bee new` project. But there are demos for model in `bee api` project. Basically, if your application is simple enough the Controller can handle everything. If there are some reuseable logic we can extract them out into a module. The Model is the result of logic extracting. Usually Model will handling some data reading and writing. Here is some code:

```
package models

import (
	"loggo/utils"
	"path/filepath"
	"strconv"
	"strings"
)

var (
	NotPV []string = []string{"css", "js", "class", "gif", "jpg", "jpeg", "png", "bmp", "ico", "rss", "xml", "swf"}
)

const big = 0xFFFFFF

func LogPV(urls string) bool {
	ext := filepath.Ext(urls)
	if ext == "" {
		return true
	}
	for _, v := range NotPV {
		if v == strings.ToLower(ext) {
			return false
		}
	}
	return true
}
```

So if your application is simple enough then you may not need models. But when your application get bigger and more reuseable code and need logic separation you must need the models. In next section we will talk about how to write the View.
