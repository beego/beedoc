---
name: Contributing
sort: 1
---

# Contributing to Beego

## Introduction

Beego is a free and open source software, which means that anyone can contribute
to its development and progress. Beego's source code is hosted on github
(https://github.com/astaxie/beego), which provides a very easy way to fork the
project and merge your contributions.

### How can I become a contributor of Beego?

You can fork, modify and then send a Pull Request to us.
We will review your code and give you feedback on your changes as soon as possible.

## Pull Requests

The process for pull requests for new features and bug fixes are not the same.

### Bug fixes

Pull requests for bug fixes do not need to create an issue first. If you have a
solution to a bug, please describe your solution in detail in your pull request.

### Documentation improvements

You can help improve the documentation by submitting pull request to the
[beedoc](https://github.com/beego/beedoc) repository.

### New features proposals

Before you submit a pull request for a new feature, you should first create an
issue with `[Proposal]` in the title, describing the new feature, as well as the
implementation approach.

Proposals will be reviewed and discussed by the core contributors, and can be
adopted or potentially rejected.

Once a proposal is accepted, create an implementation of the new features and
submit it as a pull request. If the guidelines are not followed, the pull
request will be rejected immediately.

Since Beego follows the [Git Flow](http://nvie.com/posts/a-successful-git-branching-model/)
branching model, ongoing development happens in the `develop` branch. Therefore,
please base your pull requests on the HEAD of the `develop` branch.


### The git branches of Beego

The master branch is relatively stable one where dev branch is for developers. Here is a
sample figure to show you how our branches work:

![](../images/git-branch-1.png)

For more information about the branching model: http://nvie.com/posts/a-successful-git-branching-model/
