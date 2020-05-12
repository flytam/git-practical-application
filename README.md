# git-practical-application

> 记录一些工作上和平时遇到 git 操作的问题的优雅解决方案

### Question1

公司内部有代码仓库和 github 仓库邮箱不一致。例如已经全局配置了公司内的信息

```
git config --global user.name "xxx"
git config --global user.email "xxx@tencent.com"
```

使用该配置推送到自己的 github 仓库后，那里是看不到 contribute 记录的

解决：用户.ssh 目录下新建 config 文件，配置参考如下

```config
    # tencent
       Host git.code.oa.com
       HostName git.code.oa.com
       PreferredAuthentications publickey
       IdentityFile ~/.ssh/tencent_id_rsa

    # github
       Host github.com
       HostName github.com
      PreferredAuthentications publickey
      IdentityFile ~/.ssh/id_rsa
```

IdentityFile 指定生成的每个账号 ssh key 文件路径
Host git 服务器 host
Hostname 自己定义。git@xxx

### Question2

分支开发亢余 commit 信息处理。例如自己有分支上一个小阶段 commit 一个东西，但是在合 master 的时候这些是不被允许的，需要处理

```bash
git log // 查看commit记录
```

例如，如下。需要将 3 条 step x 记录合并成一条提交。我们找到需要合并的最前那条 commit 记录的前一条（此处是 9b37084eec4e045fc0cdf218a846ec2e43a2812f）

```bash
git rebase -i 9b37084eec4e045fc0cdf218a846ec2e43a2812f
```

```bash
pick c67c886 step1
pick 0df8493 step2
pick 1ae6ab8 step 3

# Rebase 9b37084..1ae6ab8 onto 9b37084 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
```

注释已经写得很清楚。vim 下修改

```bash
pick c67c886 step1
s 0df8493 step2
s 1ae6ab8 step 3
```

保存退出，重新输入新的 commit 纪录再保存退出，这 3 条既可以合并成一条记录。如果之前已经退到远程分支仓库，可以 git push -f 进行覆盖操作

### Question3

如何优雅合并主干，遇到冲突如何处理。

合主干，假设之前在 feature/something 上开发

```bash
git checkout master
git pull --rebase orgin master
git merge --no-ff feature/something // 用非fast-forward进行合并，这样git网络比较清晰
```

如果遇到了冲突。（通常命令行和 vscode 都会有提示）

解决冲突后

```bash
git add .
git rebase --continue
```

合并主干完成

对于提 merge request 的

在 feature/something 上开发

```bash
git pull --rebase origin master
```

若冲突，解决后

```bash
git add .
git rebase --continue
git push -f origin feature/something # 由于进行了rebase，必须使用-f覆盖远程，只适用于当前分支是自己一个人开发的情况
```

提 merge request

实际情况 master 也有可能是 dev，看团队习惯

### Question4

feature 分支上开发到一半。遇到其它问题从 master 分支上切新分支处理

可以`git stash push`暂存 然后 `git stash pop`。但是个人更喜欢`commit`一个`tmp`。最后合并的时候把commit信息重写

### Question5

写了一会代码，发现自己是在本地 master 上直接写并且已经 commit 了几次了

### Question6

错误的 merge 后需要修复，这里分两种情况：

1、master 本地刚合了 feature 分支代码，但是没有推上远程。分支上的多次 commit 时间很久了，插在 master 上很多了。然后需要撤销合并。需要还原 merge 前的 master

```bash
// git log信息如下，需要撤销test分支过来的两次合并,test合并测试1 test合并测试2。
commit a0674976b94d17465eb63e799a334dd12a5ab553 (HEAD -> master)
Merge: 1f8c9ac 69e88d3
Date:   Wed Nov 7 15:56:39 2018 +0800

    Merge branch 'test'

commit 1f8c9aceb78677c2dc8169f463382fc64ec8a514
Date:   Wed Nov 7 15:56:30 2018 +0800

    master上的

commit 69e88d3fdd64547e34e248b7b77a612cae3dbe37 (test)
Date:   Wed Nov 7 15:56:05 2018 +0800

    test合并测试2

commit 1e9761a4b233137c8640129d74894d51dcb38a78
Date:   Wed Nov 7 15:55:56 2018 +0800

    test合并测试1
```

可见 a0674976b94d17465eb63e799a334dd12a5ab553 是合并过来的 commit 记录，我们直接

```bash
git reset --hard 1f8c9aceb78677c2dc8169f463382fc64ec8a514
```

即可。即回退到合并分支的上一条状态。之前我以为这样 test 合并测试那两条还在。实际上它们也在 commit 记录上没了。这种操作只适用于本地没有上远程的时候有用。

2、master 合并分支的代码已经推到了远程，需要撤销这次的提交

已经推到远程的不能像本地那么做，因为会把别人后来的代码弄没了。需要使用 revert 命令。还是上面的例子

```bash
git revert -m 1 a0674976b94d17465eb63e799a334dd12a5ab553
```

这里是放弃到合并的这次提交。注意 revert 是会生成一次新的 commit 记录，而不是把历史中的问题 commit 从 log 中清除

如果冲突，解决后

```bash
git add .
git revert --continue
```

重新推上远程，这样不会影响别人的提交，只是把自己的问题 commit 去除掉

### Question7

代码误上 master
和 6 的情况 2 类似

### Question8

仓库上一个文件需要删除

### Question9

提交了敏感文件

### Question10

追加提交。把这次的提交追加到上次的 commit。适合分支使用，push 的时候使用-f

git add .
git commit —amend

### revert 实用姿势

revert 反提交
//将 head^^^到 head 范围内的提交反转
git revert head^^^..head (3 次)
Todo 查看 revert 一些回滚某几个分支的做法
Or
git rebase -I xxx

撤销某一个文件的修改，还没有 add 的 git checkout [file]
撤销某一个文件的 add git reset HEAD [file]

Cherry-pick
将 feature 分支的某个提交合进 develop。sha 值是不一样的
git checkout develop
git log feature
git cherry-pick <C 的 SHA1>

[链接](https://ulivz.com/2018/04/12/when-you-decide-to-revert-a-merge-commit/)

未完继续....
