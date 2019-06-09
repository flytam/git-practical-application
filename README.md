# git-practical-application
> 记录一些工作上和平时遇到git操作的问题的优雅解决方案

### Question1
公司内部有代码仓库和github仓库邮箱不一致。例如已经全局配置了公司内的信息
```
git config --global user.name "xxx"
git config --global user.email "xxx@tencent.com"
```
使用该配置推送到自己的github仓库后，那里是看不到contribute记录的

解决：用户.ssh目录下新建config文件，配置参考如下
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
IdentityFile 指定生成的每个账号ssh key文件路径
Host git服务器host
Hostname 自己定义。git@xxx

### Question2
分支开发亢余commit信息处理。例如自己有分支上一个小阶段commit一个东西，但是在合master的时候这些是不被允许的，需要处理
```bash
git log // 查看commit记录
```
例如，如下。需要将3条step x记录合并成一条提交。我们找到需要合并的最前那条commit记录的前一条（此处是9b37084eec4e045fc0cdf218a846ec2e43a2812f）
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
注释已经写得很清楚。vim下修改
```bash
pick c67c886 step1
s 0df8493 step2
s 1ae6ab8 step 3
```
保存退出，重新输入新的commit纪录再保存退出，这3条既可以合并成一条记录。如果之前已经退到远程分支仓库，可以git push -f进行覆盖操作

### Question3
如何优雅合并主干，遇到冲突如何处理。

 合主干，假设之前在feature/something上开发
```bash
git checkout master
git pull orgin master
git merge --no-ff feature/something // 用非fast-forward进行合并，这样git网络比较清晰
```

如果遇到了冲突。（通常命令行和vscode都会有提示）

解决冲突后
```bash
git add .
git merge --continue
```
合并主干完成

对于提merge request的

在feature/something上开发
```bash
git pull origin master
```

若冲突，解决后
```bash
git add .
git merge --continue
git push origin feature/something
```

提merge request

实际情况master也有可能是dev，看团队习惯

### Question4
feature分支上开发到一半。遇到其它问题从master分支上切新分支处理

### Question5
写了一会代码，发现自己是在本地master上直接写并且已经commit了几次了

### Question6

错误的merge后需要修复，这里分两种情况：

1、master本地刚合了feature分支代码，但是没有推上远程。分支上的多次commit时间很久了，插在master上很多了。然后需要撤销合并。需要还原merge前的master
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
可见a0674976b94d17465eb63e799a334dd12a5ab553是合并过来的commit记录，我们直接
```bash
git reset --hard 1f8c9aceb78677c2dc8169f463382fc64ec8a514
```
即可。即回退到合并分支的上一条状态。之前我以为这样test合并测试那两条还在。实际上它们也在commit记录上没了。这种操作只适用于本地没有上远程的时候有用。

2、master合并分支的代码已经推到了远程，需要撤销这次的提交

已经推到远程的不能像本地那么做，因为会把别人后来的代码弄没了。需要使用revert命令。还是上面的例子
```bash
git revert -m 1 a0674976b94d17465eb63e799a334dd12a5ab553
```
这里是放弃到合并的这次提交。注意revert是会生成一次新的commit记录，而不是把历史中的问题commit从log中清除

如果冲突，解决后
```bash
git add .
git revert --continue
```
重新推上远程，这样不会影响别人的提交，只是把自己的问题commit去除掉


### Question7
代码误上master
和6的情况2类似

### Question8
仓库上一个文件需要删除

### Question9
提交了敏感文件

### revert实用姿势
[链接](https://ulivz.com/2018/04/12/when-you-decide-to-revert-a-merge-commit/)
未完继续....

