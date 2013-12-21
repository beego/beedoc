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
- git https is not accessable. Please config local git and close https validation:

		git config --global http.sslVerify false

- How can I install beego offline? There is no good solution now. We will creat packages for downloading and installing for every release.

# Upgrading beego

You can upgrade beego through Go command or download and upgrade from sourcecode.

- Through Go command: we recommand you using this way to upgrade beego:

		go get -u github.com/astaxie/beego
		
- Through sourcecode: visit `https://github.com/astaxie/beego` and download the sourcecode. Copy and overwrite to path `$GOPATH/github.com/astaxie/beego`. Then run `go install` to upgrade beego:

		go install 	github.com/astaxie/beego	

# The git branches of beego

The master branch is relatively stable one where dev branch is for developers. Here is a sample figure to show you how our branches work:

![](../images/git-branch-1.png)


# How can I be a contributer of beego

Beego's sourcecode is hosted on GitHub. You can fork, modify and then send a Pull Request to us. We will review your code and give you feedback of your changes as soon as possible.  
