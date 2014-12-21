---
root: true
name: Install / Upgrade
sort: 1
---

# Installing beego

You can use the classic Go way to install beego:

	go get github.com/astaxie/beego

Frequently asked questions:

- git is not installed. Please install git for your system.
- git https is not accessible. Please config local git and close https validation:

		git config --global http.sslVerify false

- How can I install beego offline? There is no good solution now. We will creat packages for downloading and installing for every release.

# Upgrading beego

You can upgrade beego through Go command or download and upgrade from source code.

- Through Go command: we recommand you using this way to upgrade beego:

		go get -u github.com/astaxie/beego

- Through source code: visit `https://github.com/astaxie/beego` and download the source code. Copy and overwrite to path `$GOPATH/src/github.com/astaxie/beego`. Then run `go install` to upgrade beego:

		go install 	github.com/astaxie/beego

**Upgrading Prior 1.0:** The API of beego is already stable after 1.0. Basically it's compatible of every upgrade. If you are still using the version lower than 1.0, you might need to change some methods and parameters based on the latest API.
