---
name: Models
sort: 4
---

# Creating models

Models are normally the best way to handle the numerous databases used in web applications.  The `bee new` project does not contain an example of models.  Demos on implementing and using models can instead be found in `bee api` projects.

The Controller can automatically handle models for simple applications.  

Larger applications with more reusable code requiring logic separation must use models.  Reusable logic can be factored out into a Model and used to handle database interactions.  The following is an example:

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

Please see [MVC Models](../mvc/model/overview.md) for the specific examples of database models and Beego's ORM framework. [The next section](view.md) will cover writing views.
