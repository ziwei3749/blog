# Git flow

- 你们工作的git规范是怎样的
- 业界Git flow是怎样的
- git pull /  git fetch /  git merge的区别

git pull 可以理解为： git fetch + git merge

因为有本地分支 、 远程本地分支 、 远程仓库的慨念


- 集中式工作流
- 功能分支工作流
- git flow
- forking工作流
- pull requests


我们用的大概就是功能分支工作流

每次git checkout -b lvziwei_branch 切换新分支后，

git push origin lvziwei_branch

然后提交一个pull request，项目领导在code review之后，没有问题会合并到master


这种方式比较灵活，问题在大型团队常常需要给不同的分支一个更加具体的角色。

Gitflow是管理功能开发、发布准备和维护的常用模式

## Gitflow

Gitflow定义了2个分支来记录项目历史版本
- master 存储正式发布的历史
- dev 作为功能开发的分支，这样也方便master分支上的所有分支一个版本号

在开发新功能时，同时切出来新的分支，但并不是从master上切出来的，而是从dev分支切出来。新功能的切换和push都跟master没有交互。


- 除了这2个长期维护的分支，还有hotfix临时分支。这个是唯一可以直接从master上切出来的分支。修复完成后，分别合并到master和dev分支





