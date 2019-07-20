---
title: A successful Git branching model
date: 2019-07-16T19:46:21.000Z
tags:
  - Git
  - 版本管理
  - branch
categories:
  - Git
  - model
---
{% cq %}

\# git同性交友指南 #

{% endcq %}

<!-- more -->

![capture_stepup1_5_6.png](https://i.loli.net/2019/07/16/5d2dc5fc1c2ac17597.png)

<br>

# 主分支(master和develop)

- - -

* **master分支**
  master分支只负责管理发布的状态,在提交时利用标签记录发布版本号
* **develop分支**
  develop分支是针对发布的日常开发分支.有合并分支的作用

<br>

# 特性分支(feature)

- - -

这个分支是针对新功能的开发，在bug修正的时候从develop分支分叉出来的。基本上不需要共享特性分支的操作，所以不需要远端控制。完成开发后，把分支合并回develop分支后发布。

<br>

# release分支

- - -

release分支是为release做准备的。通常会在分支名称的最前面加上release-。release前需要在这个分支进行最后的调整，而且为了下一版release开发用develop分支的上游分支。

一般的开发是在develop分支上进行的，到了可以发布的状态时再创建release分支，为release做最后的bug修正。

到了可以release的状态时，把release分支合并到master分支，并且在合并提交里添加release版本号的标签。

要导入在release分支所作的修改，也要合并回develop分支。

<br>

# hotFix分支

- - -

hotFix分支是在发布的产品需要紧急修正时，从master分支创建的分支。通常会在分支名称的最前面加上 hotfix-。

例如，在develop分支上的开发还不完整时，需要紧急修改。这个时候在develop分支创建可以发布的版本要花许多的时间，所以最好选择从master分支直接创建分支进行修改，然后合并分支。

修改时创建的hotFix分支要合并回develop分支。
