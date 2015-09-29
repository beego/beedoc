---
root: true
name: Установка / Обновление
sort: 1
---

# Установка BeeGo

Вы можете использовать классический путь для установки BeeGo:

	go get github.com/astaxie/beego

Часто задаваемые вопросы:

- git is not installed. Please install git for your system.
- git https is not accessible. Please config local git and close https validation:

		git config --global http.sslVerify false

- How can I install beego offline? There is no good solution now. We will creat packages for downloading and installing for every release.

# Обновление BeeGo

Вы можете обновить BeeGo используя команду Go или скачав и заменив исходный код.

- Используя Go команду: мы рекомендуем использовать этот вариант для обновления BeeGo:

		go get -u github.com/astaxie/beego

- Используя исходный код: зайдите на `https://github.com/astaxie/beego` и скачайте исходный код. Скопируйте новый исходный код и перезапишите файлы в `$GOPATH/src/github.com/astaxie/beego`. После этого запустите `go install` для обновления BeeGo:

		go install github.com/astaxie/beego

# Git-ветки в BeeGo

Ветка `master` относится к стабильному коду, а вся разработка ведется в ветке `develop`.
Вот наглядное изображение того как мы работаем с ветками:

![](../images/git-branch-1.png)


# Как я могу присоединиться к BeeGo

Исходный код BeeGo находится на GitHub. Вы можете форкнуть, доработать BeeGo и потом сделать пулл-реквест нам. Мы изучим ваш код и дадим фидбэк относительно ваших изменений так быстро, как это возможно.
