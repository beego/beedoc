---
root: true
name: Install / Upgrade
sort: 1
---

# Installing Beego

You can use the classic Go way to install Beego:

	go get github.com/astaxie/beego

Frequently asked questions:

- git is not installed. Please install git for your system.
- git https is not accessible. Please config local git and close https validation:

		git config --global http.sslVerify false

- How can I install Beego offline? There is no good solution now. We will create packages for downloading and installing for every release.

# Upgrading Beego

You can upgrade Beego through Go command or download and upgrade from source code.

- Through Go command: we recommand you use this way to upgrade Beego:

		go get -u github.com/astaxie/beego

- Through source code: visit `https://github.com/astaxie/beego` and download the source code. Copy and overwrite to path `$GOPATH/src/github.com/astaxie/beego`. Then run `go install` to upgrade Beego:

		go install 	github.com/astaxie/beego

**Upgrading Prior to 1.0:** The API of Beego is already stable after 1.0. Basically it's compatible with every upgrade. If you are still using a version lower than 1.0, you might need to change some methods and parameters based on the latest API.

