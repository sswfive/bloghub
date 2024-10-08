---
title: git使用总结
date: 2023-03-05 23:36:34
tags: 
  - Git
  - Tools
---
## Git学习资料

- [在线学习网站](https://learngitbranching.js.org/)
- [https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)
- [https://github.com/bingohuang/progit2-gitbook](https://github.com/bingohuang/progit2-gitbook)
- [git - 简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)
- [图解Git](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)

---

### status: 查看状态

```bash
git status 
git status -s
```

### add: 添加文件至缓存区

```bash
git add -i   # 交互式提交
git add .    # 添加所有变更的文件
git add xxx  # 添加指定的文件
```

### branch: 分支管理

```bash
git branch            # 查看本地所有分支
git branch -r         # 查看远程所有分支
git branch -a         # 查看本地及远程的所有分支
git branch --all      # 同上
git branch -d 分支名   # 删除本地分支

git checkout 分支         # 切换分支：

git push origin -d 分支名  # 删除远程分支

git remote show origin    # 查看远程分支和本地分支的对应关系
 
git remote prune origin   # 删除远程已经删除过的分支
```

### fetch: 更新

```bash
git fetch   # 将某个远程主机的更新，全部取回本地：

git fetch origin master   # 从远程的origin仓库的master分支下载代码到本地的origin master

git fetch origin master:temp   # 从远程的origin仓库的master分支下载到本地并新建一个分支temp
```

### diff: 对比

```bash
比较master分支和temp分支的不同
git diff temp
```

### log: 日志

```bash
git log -p master..origin/master   # 比较本地的仓库和远程仓库的区别
git log -p
```

### archive: 打包

```bash
# --format 表示打包的格式，如 zip，-v 表示对应的 tag 名，后面跟的是 tag 名，如 v0.1
git archive -v -format=zip v0.1>v0.1.zip
```

### stash:

- 修改的文件在本地，还没有add 和commit,
- 此时想切换分支，不想提交，可以先用stash缓存起来，切回来后，再还原。

```bash
git stash
git stash save "test-cmd-stash"  # 执行存储时，添加备注
git stash list                   # 查看stash了哪些存储
git stash drop                   # 删除stash暂存区
git stash pop                    # 不删除stash暂存区，可多次应用
git stash apply                  # 应用某个存储,但不会把存储从存储列表中删除
git stash clear                  # 清空栈中所有记录(删除所有缓存的stash)
```

### submodule：

- 参考1：[submodule](https://knightyun.github.io/2021/03/21/git-submodule)

- 参考2：[submodule](https://segmentfault.com/a/1190000003076028)

```bash
git submodule add git@github.com:jjz/pod-library.git pod-library
git add .gitmodules pod-ibrary
git commit -m "pod-library submodule"
git submodule init
git push
```

### subtree：

```bash
git subtree pull --prefix=<子目录名> <远程分支> <分支> 

#将 B 仓库添加为 A 仓库的一个子目录
git subtree add --prefix=SubFolder/B https://github.com/walterlv/walterlv.git master

#将 A 仓库中的 B 子目录推送回 B 仓库
git subtree push --prefix=SubFolder/B https://github.com/walterlv/walterlv.git master

#将 B 仓库中的新内容拉回 A 仓库的子目录
git subtree pull --prefix=SubFolder/B walterlv master

# 将需要分离的目录的提交日志分离成一个独立的临时版本
git subtree split -P <name-of-folder> -b <name-of-new-branch>

# example
- name: Publish
uses: hlyani/gitbook-deploy-action@master
env:
    ACCESS_TOKEN: ${{ secrets.testaction }}
    BASE_BRANCH: source
    BRANCH: master
    FOLDER: docs
git add -f $FOLDER
git commit -m "Deploying to ${BRANCH} from ${BASE_BRANCH:-master} ${GITHUB_SHA}" --quiet
git push $REPOSITORY_PATH `git subtree split --prefix $FOLDER ${BASE_BRANCH:-master}`:$BRANCH --force
```

---

## 常用命令实践

### 配置git账号

- 看看当前的配置

```bash
$ git config --list
```

- 配置用户名
  - 最好和仓库账号一致

```bash
$ git config --global user.name "<name>"
#  --global为可选参数，该参数表示配置全局信息
```

- 配置邮箱
  - 最好和仓库邮箱一致

```bash
$ git config --global user.email "<email address>"
```

- 长命令简化

```bash
$ git config --global alias.logg "log --graph --decorate --abbrev-commit --all"
# 之后就可以开心地使用 git log了
```

### 在某一个分支下的常用操作

```bash
git status
git add -a
git status
git commit -m 'xxx'
git pull --rebase
git push origin xxbran
```

### 代码提交操作

1. 文件增删改，变为已修改状态
2. git add ,变为已暂存状态

```bash
git status
git add --all # 当前项目下的所有更改
git add .  # 当前目录下的所有更改
git add xx/xx.py xx/xx2.py  # 添加某几个文件

```

3. git commit, 变为已提交状态

```bash
git commit -m "<这里写commit的描述>"
```

4. git push ，变为已推送状态

```bash
git push -u origin master # 第一次需要关联上
git push # 之后再推送就不用指明应该推送的远程分支了
git branch # 可以查看本地仓库的分支
git branch -a # 可以查看本地仓库和本地远程仓库(远程仓库的本地镜像)的所有分
```

### 代码撤销提交操作

- 已修改、但未暂存
  - diff、checkout 、clean

```bash
git diff # 列出所有的修改
git diff xx/xx.py xx/xx2.py # 列出某(几)个文件的修改

git checkout # 撤销项目下所有的修改
git checkout . # 撤销当前文件夹下所有的修改
git checkout xx/xx.py xx/xx2.py # 撤销某几个文件的修改
git clean -f # untracked状态，撤销新增的文件
git clean -df # untracked状态，撤销新增的文件和文件夹

# Untracked files:
#  (use "git add <file>..." to include in what will be committed)
#
#	xxx.py
```

- 已暂存、未提交
  - 此时，已经执行过git add, 但未执行git commit ，用于git diff已经看不到任何修改（git diff 检查的是工作区与暂存区之间的差异）

```bash
git diff --cached # 这个命令显示暂存区和本地仓库的差异

git reset # 暂存区的修改恢复到工作区
git reset --soft # 与git reset等价，回到已修改状态，修改的内容仍然在工作区中
git reset --hard # 回到未修改状态，清空暂存区和工作区

# git reset --hard 操作等价于 git reset 和 git checkout 2步操作
```

- 已提交、未推送
  - 执行完commit之后，会在仓库中生成一个版本号(hash值)，标志这次提交。之后任何时候，都可以借助这个hash值回退到这次提交。

```bash
git diff <branch-name1> <branch-name2> # 比较2个分支之间的差异
git diff master origin/master # 查看本地仓库与本地远程仓库的差异

git reset --hard origin/master # 回退与本地远程仓库一致
git reset --hard HEAD^ # 回退到本地仓库上一个版本
git reset --hard <hash code> # 回退到任意版本
git reset --soft/git reset # 回退且回到已修改状态，修改仍保留在工作区中。
```

- 已推送到远程

```bash
$ git push -f orgin master # 强制覆盖远程分支

$ git push -f # 如果之前已经用 -u 关联过，则可省略分支名

# 慎用，一般情况下，本地分支比远程要新，所以可以直接推送到远程，但有时推送到远程后发现有问题，进行了版本回退，旧版本或者分叉版本推送到远程，需要添加 -f参数，表示强制覆盖。
```

- 去掉某个commit

```bash
$ git revert <commit-hash>
# 实质是新建了一个与原来完全相反的commit，抵消了原来commit的效果
```

- reset回退错误恢复

```bash
$ git reflog #查看最近操作记录
$ git reset --hard HEAD{5} #恢复到前五笔操作
$ git pull origin backend-log #再次拉取代码
```

### 关联远程仓库

- 如果还没有Git仓库，则需要初始化git

```bash
$ git init
```

- 如果想关联远程仓库

```bash
$ git remote add <name> <git-repo-url>
# 例如 git remote add origin https://github.com/xxxxxx # 是远程仓库的名称，通常为 origin
```

- 如果想关联多个远程仓库

```bash
$ git remote add <name> <another-git-repo-url>
# 例如 git remote add coding https://coding.net/xxxxxx
```

- 如果想查看已关联的仓库（忘了关联了哪些仓库或地址）

```bash
$ git remote -v
# origin https://github.com/gzdaijie/koa-react-server-render-blog.git (fetch)
# origin https://github.com/gzdaijie/koa-react-server-render-blog.git (push)
```

- 如果有远程仓库，想需要clone到本地

```bash
$ git clone <git-repo-url>
# 关联的远程仓库将被命名为origin，这是默认的。
```

- 如果想把别人的仓库地址改为自己的

```bash
$ git remote set-url origin <your-git-url>
```

### 常用分支操作

- 如果想新建分支并切换

```bash
$ git checkout -b <new-branch-name>
# 例如 git checkout -b dev
# 如果仅新建，不切换，则去掉参数 -b
```

- 看看当前有哪些分支

```bash
$ git branch
# * dev
#   master # 标*号的代表当前所在的分支
```

- 看看当前本地&远程有哪些分支

```bash
$ git branch -a
# * dev
#   master
#   remotes/origin/master
```

- 切换到现有的分支

```bash
$ git checkout master
```

- 把dev分支合并到master分支

```bash
$ git merge <branch-name>
# 例如 git merge dev
```

- 把本地master分支推送到远程去

```bash
$ git push origin master
# 你可以使用git push -u origin master将本地分支与远程分支关联，之后仅需要使用git push即可。
```

- 远程分支被别人更新了，你想要更新代码

```bash
$ git pull origin <branch-name>
# 之前如果push时使用过-u，那么就可以省略为git pull
```

- 本地有修改，能不能先git pull

```bash
$ git stash # 工作区修改暂存
$ git pull  # 更新分支
$ git stash pop # 暂存修改恢复到工作区
```

### 常用commit操作

- 修改之前的commit
  - 将HEAD移动到需要修改的commit上

```
git rebase eb69dddd^ --interactive
```

- 追加改动到提交

```
git commit --amend
```

- 移动HEAD回到最新的commit

```
git rebase --continue
```

- 强制提交

```
git push origin master -f
```

- 找到要合并 commit 的前一个commit

```
git rebase -i sdfs1easdasd
# git rebase -i HEAD~3
```

- 将要合并的 commit 前改为 squash

```
pick asdasd Add second commit
squash asdasd Add third commit
:wq
```

- 操作失误 --abort 撤销

```
git rebase --abort
```

### 版本回退与前进操作

- 查看历史版本

```bash
$ git log
```

- 你可能觉得这样的log不好看，试试这个

```bash
$ git log --graph --decorate --abbrev-commit --all
```

- 切换到任意版本

```bash
$ git checkout a5d88ea
# hash码很长，通常6-7位就够了
```

- 远程仓库的版本很新，但你还是想用老版本覆盖

```bash
$ git push origin master --force
# 或者 git push -f origin master
```

- 觉得commit太多了，多个commit合并为1个

```bash
$ git rebase -i HEAD~4
# 这个命令，将最近4个commit合并为1个，HEAD代表当前版本。将进入VIM界面，你可以修改提交信息。推送到远程分支的commit，不建议这样做，多人合作时，通常不建议修改历史。
```

- 想回退到某一个版本

```bash
$ git reset --hard <hash>
# 例如 git reset --hard a3hd73r
# --hard代表丢弃工作区的修改，让工作区与版本代码一模一样，与之对应，--soft参数代表保留工作区的修改。
```

- 想回退到上一个版本，有没有简便方法？

```bash
$ git reset --hard HEAD^
```

- 回退到上上个版本呢？

```bash
$ git reset --hard HEAD^^
# HEAD^^可以换作具体版本hash值。
```

- 回退错了，能不能前进呀？

```bash
$ git reflog
# 这个命令保留了最近执行的操作及所处的版本，每条命令前的hash值，则是对应版本的hash值。使用上述的git checkout 或者 git reset命令 则可以检出或回退到对应版本。
```

- 刚刚commit信息写错了，可以修改吗？

```bash
$ git commit --amend
```

- 看看当前状态吧

```bash
$ git status
```

---



## gitignore文件

- [不同语言gitignore配置](https://github.com/github/gitignore)



