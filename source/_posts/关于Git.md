---
title: 关于Git
date: 2022-01-06 20:40:57
categories:
  - Code
tags: [ git ]
---


Git命令解释，假设你是<<异世界好哥哥历险记>>的编剧，咱们组使用阿里云盘对剧本压缩包保存方便随时随地可以在任意地方写作<!--more-->    
(ps:不要吐槽设定，都是剧情需要)  


### Branch 分支
每个分支就像是剧本中的不同剧情支线。
例如，煤老板A说要注资但是要改剧本里面加个女二，这个剧情支线可以作为分支，分支名可以叫"XXX无惨"，再比如"送审13.0.23.20240123"等等。

```
$ git branch -a
* master
  songshen13.0.23.20240123
  remotes/origin/HEAD -> origin/master
  remotes/origin/xxxwucan
  remotes/origin/master
```

### 
