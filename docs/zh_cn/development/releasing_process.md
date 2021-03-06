# BFE版本发行规范

## BFE分支规范

BFE开发过程使用[git-flow](http://nvie.com/posts/a-successful-git-branching-model/)分支规范，并适应github的特性做了一些区别。

* BFE的主版本库遵循[git-flow](http://nvie.com/posts/a-successful-git-branching-model/)分支规范。其中:
	* `master`分支为稳定(stable branch)版本分支。每一个`master`分支的版本都是经过单元测试和回归测试的版本。
	* `develop`分支为开发(develop branch)版本分支。每一个`develop`分支的版本都经过单元测试，但并没有经过回归测试。
	* `release/版本号`分支为每一次Release时建立的临时分支。在这个阶段的代码正在经历回归测试。

* 其他用户的fork版本库并不需要严格遵守[git-flow](http://nvie.com/posts/a-successful-git-branching-model/)分支规范，但所有fork的版本库的所有分支都相当于特性分支。
	* 建议，开发者fork的版本库使用`develop`分支同步主版本库的`develop`分支
	* 建议，开发者fork的版本库中，再基于`develop`版本fork出自己的功能分支。
	* 当功能分支开发完毕后，向BFE的主版本库提交`Pull Reuqest`，进而进行代码评审。
		* 在评审过程中，开发者修改自己的代码，可以继续在自己的功能分支提交代码。

* BugFix分支也是在开发者自己的fork版本库维护，与功能分支不同的是，BugFix分支需要分别给主版本库的`master`、`develop`与可能有的`release/版本号`分支，同时提起`Pull Request`。


## BFE版本发行

BFE使用git-flow branching model做分支管理，使用[Semantic Versioning](http://semver.org/)标准表示BFE版本号。

BFE每次发新的版本，遵循以下流程:

1. 从`develop`分支派生出新的分支，分支名为`release/版本号`。例如，`release/0.10.0`
1. 将新分支的版本打上tag，tag为`版本号rc.Patch号`。第一个tag为`1.0.0rc1`，第二个为`1.0.0rc2`，依次类推。
1. 对这个版本的提交，做如下几个操作:
	* 编译这个版本的Docker发行镜像，发布到dockerhub。如果失败，修复Docker编译镜像问题，Patch号加一，返回第二步
	* 编译这个版本的Ubuntu Deb包。如果失败，修复Ubuntu Deb包编译问题，Patch号加一，返回第二步。
	* 使用Regression Test List作为检查列表，测试Docker镜像/ubuntu安装包的功能正确性
		* 如果失败，记录下所有失败的例子，在这个`release/版本号`分支中，修复所有bug后，Patch号加一，返回第二步

1. 第三步完成后，将`release/版本号`分支合入master分支，并删除`release/版本号`分支。将master分支的合入commit打上tag，tag为`版本号`。同时再将`master`分支合入`develop`分支。最后删除`release/版本号`分支。
1. 编译master分支的Docker发行镜像，发布到dockerhub。编译ubuntu的deb包，发布到github release页面
1. 协同完成Release Note的书写

需要注意的是:
* `release/版本号`分支一旦建立，一般不允许再从`develop`分支合入`release/版本号`。这样保证`release/版本号`分支功能的封闭，方便测试人员测试PaddlePaddle的行为。
* 在`release/版本号`分支存在的时候，如果有bugfix的行为，需要将bugfix的分支同时merge到`master`, `develop`和`release/版本号`这三个分支。
