# git-practical-application
> 记录一些工作上和平时遇到git操作的问题的优雅解决方案

### Question1
公司内部有代码仓库和github仓库邮箱不一致。例如已经全局配置了公司内的信息
```
git config --global user.name "xxx"
git config --global user.email "xxx@tencent.com"
```
使用该配置推送到自己的github仓库后，是个人那里是看不到contribute记录的


### Question2
分支开发亢余commit信息处理。例如自己有分支上一个小阶段commit一个东西，但是在合master的时候这些是不被允许的，需要处理

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

