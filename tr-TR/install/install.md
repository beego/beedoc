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

- How can I install Beego offline? There is no good solution for now. We will create packages for downloading and installing for future releases.

# Upgrading Beego

You can upgrade Beego through Go command or download and upgrade from source code.

- Through Go command (Recommended):

		go get -u github.com/astaxie/beego

- Through source code: visit `https://github.com/astaxie/beego` and download the source code. Copy and overwrite to path `$GOPATH/src/github.com/astaxie/beego`. Then run `go install` to upgrade Beego:

		go install 	github.com/astaxie/beego

**Upgrading Prior to 1.0:** The API of Beego is stable after 1.0 and compatible with every upgrade. If you are still using a version lower than 1.0 you may need to configure your parameters based on the latest API.

