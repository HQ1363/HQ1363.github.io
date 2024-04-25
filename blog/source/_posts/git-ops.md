---
title: git操作指南   
date: 2021-08-02 14:36:50   
tags: git   
categories: Git   
excerpt: git已然成为开发的必备技能了，来看看有哪些相见恨晚的小技巧吧，可别说我没告诉你哦.
---

## Git SubModule

### 常见操作
```shell
$ git submodule init && git submodule update   # 将子模块下载到本地
$ git clone https://github.com/xx/xx.git --recursive  # 此命令可一次性下载好主库和子库
$ git submodule sync  # 修改.gitmodule文件后，用此命令同步子模块信息
$ git submodule add https://github.com/xxxx/xxxxx.git  # 添加submodule
```

### 批量操作submodule
```shell
$ git submodule foreach <command>
比如:
$ git submodule foreach git checkout master
$ git submodule foreach git submodule update
```

### 删除git submodule
```shell
$ git add .gitmodules        # 第一步
$ git rm --cached submodule_name    # 第二步
$ rm -rf submodule_name   # 第三步：删除submodule目录
# 第四步：编辑.gitmodules文件, 移除对应的submodule信息
# 第五步：编辑.git/modules文件, 移除对应的submodule信息
# 第六步：编辑.git/config 移除对应的submodule信息
# ===== 上述方式不行，可尝试下述
$ git submodule deinit <submodule-name> # 新版git
# ===== 上述方式不行，可尝试下述
$ git rm <submodule-name> # 旧版git
```

### 子仓如何与远端保持同步
```shell
# 子仓的更新，是单独的，需要进入子仓目录，手动与远端同步，例如：
$ cd sub-dir && git fetch origin master && git rebase origin/master
# 完成同步后，需要在主仓下提交子仓的改动，以保存主仓对子仓的最新引用
```

## Git Ops

### 查看last commitId
```shell
$ git log -1 --pretty=format:%H # 完整的
# 输出 7b6b2803d2b7135b239d062847816e55a810371e
$ git log -1 --pretty=format:%h # 前7位
# 输出 7b6b280
```

### 查看某次commit的内容
```shell
$ git show commitId
$ git show <commitid> --stat
```

### 查看目录的diff信息
```shell
$ git diff <dir-name>
```

### diff迁移
```shell script
$ git diff > diff.patch
$ git apply --stat diff.patch
$ git apply --check diff.patch
$ git apply diff.patch
```

### 查看log的其他信息
```shell
git --no-pager log -2 --author="HQ" --pretty=format:"%h"
```

### 指定tag/branch克隆仓库
```shell
$ git clone -b <tag-name|branch-name> xxxxxxxxx.git
```

### 获取指定tag下的仓库地址
```shell
# 浏览器地址上，替换对应的tag名称即可
# https://gitlab.com/gitlab-org/gitaly/-/tree/v13.12.8?ref_type=tags  # 官方gitaly仓库地址
```

### 查看分支合并图
```shell
$ git log --graph
$ git log --graph --since="2024-03-05" --until="2024-03-08"   # 查询某个区间下的commit图谱
```

### 查看标签信息
```shell
$ git show <tag-name>
```

### 查看git命令操作历史
```shell
$ git reflog
```

### git rebase发生冲突怎么办
```shell
$ git rebase --skip  # 抛弃本地的 commit，采用远程的 commit。慎用：因为你本地的修改都会失去
$ git rebase --abort # 终止此次 rebase 操作
$ git rebase --continue # 手动处理冲突的文件：执行git add .，再 git rebase --continue，反复操作直到解决完所有冲突，并合并到分支上。
# 切记，整个rebase解决冲突的过程中，都不需要自己去单独执行commit动作
```

### 删除本地认证
```shell
$ git config --global --unset credential.helper  # --system or --local
$ git config credential.helper store  # window的可能需要手动找到git的凭证删掉，见下图
```
![](git_identity.png)

### HTTP认证方式更改为SSH
```shell
$ git config --global url.ssh://git@github.com/.insteadOf https://github.com/
# 执行上述命令后，可以查看下全局的.gitconfig配置(cat ~/.gitconfig)如下：
[url "ssh://git@gitlab.sss.com/"]
	insteadOf = https://gitlab.sss.com/
[url "ssh://git@pkg.sss.com/"]
	insteadOf = https://pkg.sss.com/
[http]
	sslVerify = false
[url "ssh://git@github.com/"]
	insteadOf = https://github.com/
# 完成替换后，记得添加ssh public key到gitlab profile配置里
# 删除Local认证
$ git config --local --unset credential.helper
# 删除全局认证
$ git config --global --unset credential.helper
# 删除系统认证
$ git config --system --unset credential.helper
# 设置记住密码（默认15分钟）：
$ git config --global credential.helper cache
# 如果想自己设置时间，可以这样做
$ git config credential.helper 'cache --timeout=3600'
# 永久存储密码
$ git config --global credential.helper store
# 清理缓存的账号密码
$ git credential-manager uninstall
```

### 如何知道分支是从哪拉出来的
```shell
$ git reflog --date=local | grep 本地分支名       # 仅限本地创建的分支
$ git log --oneline --decorate --graph --all    # 看的迷糊，看不懂
# 可借助sourceTree等工具查看，比较稳
```

### git的两点diff和三点diff区别是啥
```shell
# 三点diff会找到两边的共同祖先，然后拿到祖先到最新版本的差异，可能会有以下应用场景：
# 1、feature分支提交了，很多次，我想知道，我的feature相比于主干分支(master)的所有改动内容，然后codereview
```

### git远端分支被删除后，还可以恢复吗？
```shell
# 1、如果本地或者其他地方，有备份的话，就直接重推一下就好了
# 2、如果存有分支被删除时的，最新sha，那可以继续sha来创建分支恢复
注意：通过sha的方式恢复，要求分支的最新commit，必须已合入目标分支（主干分支），否则无法恢复分支。
git checkout -b branch/xxx <commitID>，对应gitlab api创建分支时，指定commit
----   
如果你误删除了 Git 的远程分支，有可能找到该提交的 SHA1 校验值并恢复。
首先，你需要找到该分支最后一个提交的 SHA1 校验值，使用 git reflog 命令可以查看历史的提交记录：
git reflog
在历史记录中找到你误删除的分支的最后一次提交，记下其 SHA1 校验值。
然后，从这个提交创建一个新的分支：
git branch <branch_to_recover> <SHA1>
最后，将恢复的分支推送到远程仓库：
git push origin <branch_to_recover>
这样就实现了误删的远程分支的恢复。
请注意，git reflog 命令仅会记录本地的 Git 操作，如果你在删除远程分支后没有在本地进行过任何 Git 操作，可能无法用这个办法找到该删除的分的 SHA1 校验值。同时，这种方法只能用于恢复误删的最后一次提交，如果在删除分支后做了新的提交，这种方法将无法找回这些新的提交。
```

### go get -v gitlab.com/oxxx/yyyyy 报错
```
Fetching https://xxx.local/a/b/project?go-get=1
Parsing meta tags from https://xxx.local/a/b/project?go-get=1 (status code 200)
get "xxx.local/a/b/project": found meta tag main.metaImport{Prefix:"xxx.local/a/b", VCS:"git", RepoRoot:"https://xxx.local/a/b.git"} at https://xxx.local/a/b/project?go-get=1
get "xxx.local/a/b/project": verifying non-authoritative meta tag
Fetching https://xxx.local/a/b?go-get=1
Parsing meta tags from https://xxx.local/a/b?go-get=1 (status code 200)
xxx.local/a/b (download)
package xxx.local/a/b/project: /home/user/go/src/xxx.local/a/b exists but /home/user/go/src/xxx.local/a/b/.git does not - stale checkout?
```
亦或者是如下的报错：
```
go module xxxx.xxx.com/ssss/bbbb/zzzz: git ls-remote -q origin in xxxxx: exit status 128:
   remote: The project you were looking for could not be found or you don't have permission to view it.
      fatal: repository 'xxxx.xxx.com/ssss/bbbb.git' not found
```
解决办法如下：
```
编辑 vim ~/.netrc 添加：
machine xxx.yyy.com
login username
password <personal-token>
```
详情可见：https://gitlab.com/gitlab-org/gitlab-foss/-/issues/30785

### 一个分支上的几个commit的代码不再需要了，如何处理?
方式一：
```shell
$ git reset --hard <commit-id>
```
方式二：
如果你有一个分支上的几个 commit 的代码不再需要了，你可以使用以下步骤来移除这些不需要的代码：
首先，确保你当前位于包含这些不需要的 commit 的分支上。你可以使用 git branch 命令查看当前所在的分支，并使用 git checkout 命令切换到目标分支。
运行以下命令来执行交互式的 rebase 操作：
`git rebase -i commit_hash`
这里的 commit_hash 是你要移除的最早的 commit 的哈希值。
例如，如果你要移除从 commit_hash 开始的三个 commit，你可以使用：
`git rebase -i commit_hash~3`
这个命令将打开一个文本编辑器，显示所有要进行 rebase 的 commit。
在打开的文本编辑器中，将要移除的 commit 的关键词由 pick 改为 drop 或者直接删除对应的行。例如，如果要移除第二个 commit，可以修改为：
```
pick commit_hash
drop commit_hash
pick commit_hash
...
```
保存并关闭文本编辑器。
Git 将会执行 rebase 操作，并移除指定的 commit。如果有冲突发生，Git 会暂停并提示你进行处理。根据提示解决冲突，并继续 rebase 的过程。
在 rebase 完成后，使用 git log 命令查看提交历史，你会发现已经成功移除了指定的 commit。
请注意，执行 git rebase 命令会修改提交历史，如果这些 commit 已经被推送到远程仓库，你可能需要使用 git push --force 强制推送更新后的提交历史。然而，强制推送会覆盖远程仓库的提交历史，所以请确保在删除之前与团队成员进行适当的沟通和讨论。
同时，请确保在执行这些操作之前，你已经备份了重要的代码或者创建了一个分支来保存这些代码，以防止意外丢失。

方式三：
git revert 命令可以一次撤销多个 commit。你可以使用以下步骤来一次性撤销多个 commit：
首先，确保你当前位于包含要撤销的 commit 的分支上。
使用 git log 命令查看提交历史，找到你要撤销的 commit 的哈希值。记录下这些 commit 的哈希值。
运行以下命令来撤销多个 commit：
`git revert --no-commit commit_hash1..commit_hash2`
这里的 commit_hash1 和 commit_hash2 分别是要撤销的 commit 的起始和结束哈希值。
例如，如果你要撤销从 commit_hash1 到 commit_hash2 这两个 commit，你可以使用：
`git revert --no-commit commit_hash1..commit_hash2`
这个命令会创建一个新的撤销 commit，同时撤销指定范围内所有的 commit。注意，--no-commit 选项会告诉 Git 不要自动提交撤销 commit。
在撤销 commit 之后，你可以使用 git status 命令来查看撤销的更改。
如果一切正常，使用以下命令来提交撤销 commit：
`git commit -m "Revert multiple commits"`
这个命令将会创建一个新的 commit，包含了所有撤销的更改。
请注意，每个被撤销的 commit 都会创建一个撤销 commit，因此最终会创建多个新 commit。在撤销多个 commit 之前，请确保备份了重要的代码或者创建了一个分支来保存这些代码。
git revert 命令也可以撤销多个非连续的 commit。以下是使用 git revert 撤销多个非连续 commit 的步骤：
首先，确保你当前位于包含要撤销的 commit 的分支上。
使用 git log 命令查看提交历史，找到你要撤销的每个 commit 的哈希值。记录下这些 commit 的哈希值。
运行以下命令来撤销每个 commit：
```shell
git revert --no-commit commit_hash1
git revert --no-commit commit_hash2
...
```
这里的 commit_hash1，commit_hash2，等等是要撤销的每个 commit 的哈希值。
例如，如果你要撤销两个 commit，分别是 commit_hash1 和 commit_hash2，你可以使用：
```shell
git revert --no-commit commit_hash1
git revert --no-commit commit_hash2
```
这些命令会为每个 commit 都创建一个新的撤销 commit。注意，--no-commit 选项会告诉 Git 不要自动提交撤销 commit。
在撤销 commit 之后，你可以使用 git status 命令来查看撤销的更改。
如果一切正常，使用以下命令来提交撤销 commit：
`git commit -m "Revert multiple commits"`
这个命令将会创建一个新的 commit，包含了所有撤销的更改。
