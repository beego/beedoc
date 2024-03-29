---
name: Контроллер
sort: 3
---

# Логика контроллера

В предыдущем разделе мы изучили, как распределять запросы пользователей по контроллерам. В этом разделе мы изучим, как писать контроллер. Давайте начнём с этого кода:

```
package controllers

import (
        "github.com/astaxie/beego"
)

type MainController struct {
        beego.Controller
}

func (this *MainController) Get() {
        this.Data["Website"] = "beego.vip"
        this.Data["Email"] = "astaxie@gmail.com"
        this.TplNames = "index.tpl"
}
```

В начале мы создаём `MainController`, который содержит анонимное поле структуры типа `beego.Controller`. Это называется структурной композицией, и это - способ, которым в Go имитируется наследование. Это означает, что `MainController` автоматически получает все методы `beego.Controller`.

`beego.Controller` имеет множество методов, таких как `Init`, `Prepare`, `Post`, `Get`, `Delete` и `Head`. Мы можем переписать эти функции путём своей реализации их. В данном случае мы переписали метод `GET`.

Мы говорили о том, что Beego - REST-фреймворк, так что наши запросы будут запускать по умолчанию метод `req.Method`. Например, если браузер посылает запрос `GET`, выполнится метод `Get` из `MainController`. Поэтому метод `Get` и его логика, определённая нами выше, будут выполнены.

Логика нашего `Get`-метода просто в выводе некоторых данных. Мы можем получить наши данные многими способами и сохранить их в `this.Data`, который является словарём map[string]interface{}. Мы можем назначить здесь любые данные любого типа. В нашем случае мы сохранили две строки.

Последнее, что мы должны сделать - отображение шаблона. `this.TplNames` определяет шаблон, который будет отображён: У нас это шаблон `index.tpl`. Если вы не укажите имя шаблона, он установится по умолчанию как `controller/method_name.tpl`. Например, в нашем случае контроллер будет пытаться найти `maincontroller/get.tpl`.

BeeGo автоматически вызовет `Render`-функцию (которая включена `beego.Controller`) если вы установили шаблон, так что вам не нужно делать это вручную.

Это было только краткое введение. Обратитесь к разделу про контроллеры во введении в MVC, чтобы читать дальше. В дальнейшем мы обсудим с вами, как писать модели.
