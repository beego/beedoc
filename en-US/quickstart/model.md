---
name: Models
sort: 4
---

# Creating models

We know we are using databases a lot in web applications so models are the  usual way to handle these kind of jobs. There is no demo on models in our `bee new` project, but there are demos on implementing and using models in the `bee api` project. Basically, if your application is simple enough the Controller can handle everything. However, if there is some reuseable logic we can factor it out into a module. The Model is the result of such logic extraction. Usually the model will be handling some data reading and writing. Here is an example:

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

So if your application is simple enough then you may not need models at all. But when your application get bigger and you want more reuseable code and need logic separation you must use models. In the next section we will talk about how to write the View.
