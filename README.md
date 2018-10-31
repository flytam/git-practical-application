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
如何优雅合并主干，遇到冲突如何处理。（重点问题）

### Question4
feature分支上开发到一半。遇到现网问题紧急处理

### Question5
写了一会代码，发现自己是在本地master上直接写并且已经commit了几次了

### Question6
master本地刚合了feature分支代码，分支上的多次commit时间很久了，插在master上很多了。然后合了的master不能发，又要去修现网问题。需要还原merge前的master

### Question7
代码误上master

### Question8
仓库上一个文件需要删除

### Question9
提交了敏感文件

未完继续....

