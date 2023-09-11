---
title: 回退git远程提交
date: 2022-01-06 20:40:57
categories:
  - Code
tags: [ git ]
---


严格意义上来说，我们并不期待撤销一个已经提交的commit（虽然我们可以）  
下面给出4种解决方式，各有优缺点<!--more-->  

## 使用`git revert`， 推荐但是比较麻烦
> 推荐使用`git revert`实现,将某一个提交生成一个完全相反的改动并提交


假设我们当前在main分支上commit如下， 当前在0223037e，期待回退到8266909f上
```
commit 0223037e
commit 37a67ef8
commit 8266909f
```


- 不包含merge提交  
此时我们只用依次回退0223037e以及37a67ef8即可
```
$ git revert --no-commit 0223037e
$ git revert --no-commit 37a67ef8

# or 等价于
$ git revert --no-commit 37a67ef8...0223037e
```

之后就正常commit+push即可


- 包含merge提交  
现在假定0223037e为merge提交，由dev merge main，我们当前在main分支上    
此时当我们执行git revert --no-commit 0223037e会报错 `0223037e is a merge but no -m option was given.`  
此时命令应该改为`git revert --no-commit 0223037e -m 1`  
剩下的逻辑跟不包含merge的提交一致即可  
`-m`参数后面是parentNumber, 官方文档是说 `This option specifies the parent number (starting from 1) of the mainline and allows revert to reverse the change relative to the specified parent.`   
翻译一下就是，当你在 B 分支上把 A merge 到 B 中，那么 B 就是merge commit 的 parent1，而 A 是 parent2。



## 直接git reset --hard， 省力但是如果有很多人和你协同工作的话， 藏好他们的刀  
> --hard 之后修改了git commit的index，因此期待提交时必须要通过git push -f,  
但是一旦你强行将push上去了之后，这回导致其他获取项目的同事需要面临无法正常git pull或者无法正常git push的情况



## git diff --binary HEAD commit_sha_you_want_to_revert_to | git apply 
> 存在缺陷部分无法通过git diff获取变更的无法使用该命令例如png

## 手动通过git reset --hard拿到一份想要的内容，git pull 拉去到最新的 然后将之前的内容覆盖回来
朴实无华


# 参考：  
https://git-scm.com/docs/git-revert#git-revert--mparent-number
https://stackoverflow.com/questions/1463340/how-to-revert-multiple-git-commits
https://juejin.cn/post/6844903590545506312

