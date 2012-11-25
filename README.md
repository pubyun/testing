# Gerrit和git的流程

## 资源

* 公司git服务器

<ssh://git@git.bitcomm.cn:2203/testing.git>

注意将testing换成对应的项目

* 公司gerrit评审系统

<http://review.bitcomm.cn>

* jenkins CI服务器

<http://jenkins.bitcomm.cn>

* github上的testing项目, 从gerrit自动同步到github

<https://github.com/pubyun/testing>

## 参考资料：

### 有关git的参考资料

* pro git中文版, 最好的git书籍

<http://git-scm.com/book/zh>

* 图解git

<http://marklodato.github.com/visual-git-guide/index-zh-cn.html>

* git交互式学习

<http://try.github.com/levels/1/challenges/1>

* Git分支管理策略

<http://blog.jobbole.com/23398/>

* type3的关于git和Gerrit资料

<http://wiki.typo3.org/Git_Gerrit>

* egit 和 gerrit的实践应用

<http://wiki.typo3.org/Contribution_Walkthrough_with_EGit>

### 有关gerrit 的参考资料

* <http://gerrit-documentation.googlecode.com/svn/Documentation/2.5/intro-quick.html>
* <http://www.slideshare.net/lucamilanesio/gerrit-code-review>
* <http://qt-project.org/wiki/Gerrit-Introduction>
* <http://wiki.typo3.org/Contribution_Walkthrough_with_CommandLine>
* <http://wiki.whamcloud.com/display/PUB/Using+Gerrit>
* <http://wiki.openstack.org/GerritWorkflow>
* <http://wiki.openstack.org/GerritJenkinsGithub>
* <http://www.itk.org/Wiki/ITK/Git/Develop#Create_a_Topic>
* <http://openstack-ci.github.com/publications/jenkins/>
* <http://source.android.com/source/life-of-a-patch.html>
* <http://www.infoq.com/cn/articles/Gerrit-jenkins-hudson>

## 注册帐号

在<http://review.bitcomm.cn>注册用户，采用的是openid统一认证，直接使用公司的google邮箱就可以了。

注册的时候，注意:

* 设置自己的用户名，必须为英文或者数字，这个后面授权需要用到，比如我选择的是 refactor
* 导入自己的公钥，这个在 git 进行 review操作的时候，授权需要用到

注册完了以后，可以看到 testing 项目，可以进行练习和测试。其余的项目，联系项目经理进行授权，将下面的信息发送给项目经理：

注册的邮箱、ssh公钥和对应的项目


## 工具安装

### Windows

* 安装git 工具

由于目前git的命令行功能和稳定性比图形界面的egit等强，大家在代码递交时，还是使用Windows或者Linux的命令行，windows下可以采用：

windows下的命令行工具：  
<http://msysgit.github.com>  
windows下的图形工具：  
<https://code.google.com/p/tortoisegit/>

* 下载安装windows版本的python，一般可以选用2.7.3

<http://www.python.org/download/releases/2.7.3/>

* 下载安装windows版本的setuptools

<http://pypi.python.org/pypi/setuptools>

* 下载、解压、安装 pip 软件

<http://pypi.python.org/pypi/pip#downloads>

进入解压以后的目录，然后执行安装：

    python setup.py install

* 安装 fabric 和 git-review

    pip install fabric git-review

### Ubuntu

    sudo aptitude install python-pip
    sudo pip install fabric git-review
    
### CentOS

    sudo yum -y install python-setuptools  python-devel python-pip
    sudo pip-python install fabric git-review

## 导出项目，初始设置

配置递交代码人信息：

    git config --global user.name "Peng Yong"
    git config --global user.email ppyy@pubyun.com

从git取出代码：

    git clone ssh://git@git.bitcomm.cn:2203/testing.git

注意选用自己需要的项目名称替代 testing

加入 review 评审代码的地址:

    git remote add review ssh://refactor@review.bitcomm.cn:29418/testing.git
    scp -P 29418 refactor@review.bitcomm.cn:hooks/commit-msg .git/hooks/

注意:

* 选用自己需要的项目名称替代 testing
* 使用自己的用户名替代 refactor

## 日常开发流程

当需要修改一个bug，或者开发新功能时, 需要分几部：

获取最新的代码, 防止和他人冲突

    git remote update
    git checkout master
    git pull origin master

每一个开发（bug，feature），都创建一个独立的分支，不要在 master上做，一般一个单元开发创建一个分支，互相不混淆。否则如果评审不通过，重新修改会麻烦：

    git checkout -b bug/typefix
    git checkout -b feature/login_module

修改代码，然后检查代码：

    git diff

本地递交代码:

    git commit –a
    
如果要放弃commit，使用 ：
	
	git reset --soft HEAD^

注意，看一下这些文件是否都要递交，写好简要、清晰的递交日志。

上传代码，等待评审：

    git push review HEAD:refs/for/master

如果代码审查通过，合并完成以后，可以删除这个分支：

    git checkout master
    git branch -d bug/typefix

## 如果评审不通过，需要再次修改代码，则继续在原来的分支修改代码

切换到原来的开发分支：

    git checkout bug/typefix

修改代码…

然后递交（注意，一定使用 amend选项，这样可以继续递交在原来的review单号上）

    git commit -a --amend
    git push review HEAD:refs/for/master

如果代码审查通过，合并完成以后，可以删除这个分支：

    git checkout master
    git branch -d bug/typefix

## 审批通过以后，gerrit提示有冲突怎么办

参见合并的参考文档：

* <http://unethicalblogger.com/2010/04/02/a-rebase-based-workflow.html>
* <http://www.mediawiki.org/wiki/Gerrit/resolve_conflict>

冲突产生，是由于两个开发人员，修改了同一个文件。解决办法:

    git fetch origin
    git rebase origin/master

git合并能力很强，一般的冲突上面可以自动解决了。如果冲突在同一个地方，需要手工解决。这个情况，请联系资深工程师帮助一起解决。需要用编辑器修改相应文件, 然后标志这个文件冲突解决，继续rebase：
	
	标记冲突：
		git add -u
		
    git add file_modified
    git rebase --continue

如果有多个文件同时冲突，需要运行上述命令多次。

最后递交审查：

    git commit -a --amend
    git push review HEAD:refs/for/master

## 私有分支的使用

私人分支的应用场景
    
1. 在正式递交代码之前，开发调试往往需要和同事协同开发
2. 需要部署到测试机上进行测试, 但又不想递交到master
3. 个人在公司、家里、工地进行代码的同步
4. 个人需要开发一个较复杂的功能，在递交评审之前，需要频繁递交代码, 并且为了代码安全性，需要上传到远程备份

这时就需要使用gerrit的私有分支, 具体的步骤是：

* 私有分支的命名：

    dev/username/branchname
    dev 私有开发分支的前缀，必须是 dev
    username 你的 gerrit系统的用户名
    branchname 你创建的私有分支名字，可以创建多个分支

* 创建本地的私有分支

    git checkout -b dev/refactor/fabric

* 开发，然后递交到本地

* 上传到 gerrit，并自动同步到 公司的官方git源（一般叫origin)

    git push gerrit HEAD:dev/refactor/fabric

* 其他同事切换到这个私有分支，并且获取刚才的修改

    git fetch origin
    git checkout dev/refactor/fabric
    git pull

在这里可以进行修改和递交，实现协同开发和代码同步

* 这个分支，经过测试以后，准备递交评审之前，一般使用 rebase 进行适当的合并, 然后递交

    git rebase -i HEAD~10
    git review

* 如果这个分支不再需要了，删除这个分支(本地和远程)

    git push gerrit :dev/refactor/fabric
    git checkout master
    git branch -d dev/refactor/fabric

## fabric的使用

为了提供工作效率，降低出错的几率，重复性的工作，尽量使用 fabric自动部署工具：

* 生产机上的部署

    fab

* 测试分支的部署

    fab branch:dev/refactor/fabric deploy

## 版本管理规则

### 一般线上运行的系统，只采用一个 master 主线的方式进行管理

也就是开发人员的代码，通过评审以后，直接merge到master分支；master分支也是生产机上运行的代码。这就要求有质量控制过程，防止错误导致系统的严重错误。

* 递交的代码，要是一个原子操作，具有高内聚性，即一次递交的所有代码完成且仅完成一个功能，联系紧密，缺一不可
* 递交的代码，要具有上线标准，要是一个可运行的合格的代码
* 评审一般都必须至少一个以上的人评审过，最好是同组的开发人员评审，项目经理或者资深开发人员审批。评审的过程，也是结对编程的思想，可以互相熟悉代码，互相学习提高，便于统一代码风格，提高代码质量。

### 如果一个 feature 是一个需要较长时间开发，比如增加一个短信验证的功能，需要一周时间

在自己的这个开发分支内，可以采取小步快跑的方式不断递交到本地的git。在开发完成以后，在上传到 review服务器进行评审前，需要使用 rebase 命令，将这些多次递交适当进行合并然后上传。

记住，每一次递交在Gerrit评审服务器上，都必须要单独评审审批，如果一个功能有很多小的修改组成，这些小的修改可以适当合并，具体方法是：

    git checkout master
    git pull origin master
    git checkout your-feature-branch
    git rebase -i master

### 如果需要进行改版，会延续几个月的大量改进

需要在 gerrit 评审系统内创建新的开发分支，比如devel分支，而不使用 master 进行管理，以便进行代码的隔离。改版完成以后，将代码从develp合并到master分支。

这时，改版递交代码和评审，都在develop分支进行，上传命令采用；

    git push review HEAD:refs/for/develop

生产机上任然使用 master分支。 这时，一般性的功能改进和bug修正，仍然递交到 master 分支。并且定时从 master merge到develop分支，以便减少冲突，降低以后 develop合并到 master的难度。


## FAQ
* 评审没有通过，但是原有开发分支已经删除，如何恢复这个分支，继续修改代码

在gerrit上找到这个评审单，定位到响应的patchset，里面有获取代码、恢复分支的方式链接, 比如：

    git fetch ssh://refactor@review.bitcomm.cn:29418/testing refs/changes/07/7/1 && git checkout FETCH_HEAD
    git checkout -b my_branch

* 如何获取通知邮件

对于关注的项目，可以获取通知邮件，方法是选择菜单：

    Settings - Watched Projects

然后使用 Browse 按钮选择响应的项目，然后选择“Email Notifications”的类型，一般全部打勾

* 如何gerrit评审系统的键盘快捷键

gerrit是google开发的软件，同样支持Google风格的快捷键，类似Gmail的风格。使用快捷键，可以提高阅读和评审的速度。使用问号 "?" 可以寻求快捷键的帮助。

* 提示"Missing Change-Id in commit message"

这是由于没有下载hook, 生成Change-Id。这个Change-ID是gerrit一个必须的重要标志，多次修改同一个问题的时候，会对应到一个单子。解决方法。

    scp -P 29418 refactor@review.bitcomm.cn:hooks/commit-msg .git/hooks/

然后再次递交：

    git commit -a --amend
    git push origin HEAD:refs/for/master

* Backporting a change to other branches

比如，一个master分支，一个release分支，递交进入master的分支中，有某个bugfix或者新特性需要合并回release分支：

<http://wiki.typo3.org/Contribution_Walkthrough_with_CommandLine>

    git fetch --all
    git checkout -b rfc/4-4/1234 origin/<release-branch>
    git cherry-pick <revision-id>

去掉Change-Id, Reviewed-*, Tested-by等日志信息

    git commit -a --amend
    git push origin HEAD:refs/for/<release-branch>

## 一些场景

### 前端工程师如何递交代码？

如果和程序员一起配合做一个功能，那么将代码交给程序员，开发、测试以后，一起作为一个原子操作递交。

如果是前端修改一个错误或者改进，不需要和程序员配合，则直接递交。

