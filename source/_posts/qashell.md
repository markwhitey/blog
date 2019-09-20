---
title: 记录Zookeeper脚本出现的无法启动集群问题
mathjax: false
date: 2019-09-20 10:40:10
tags:
- Big Data
- Linux
categories:
- Big Data
top:
photo:
---



{% cq %}

**脚本启动zookeeper集群需要的特别配置**

{% endcq %}

<!-- more -->

<br>

# Question

----

>  设置zookeeper集群启动脚本的时候无法正常启动

```shell
#! /bin/bash

case $1 in
"start"){
	for i in linux-1 linux-2 linux-3
	do
		ssh $i "/opt/module/zookeeper-3.4.10/bin/zkServer.sh start"
	done
};;
"stop"){
	for i in linux-1 linux-2 linux-3
	do
		ssh $i "/opt/module/zookeeper-3.4.10/bin/zkServer.sh stop"
	done
};;
"status"){
	for i in linux-1 linux-2 linux-3
	do
		ssh $i "/opt/module/zookeeper-3.4.10/bin/zkServer.sh status"
	done
};;
esac
```



<br>

# WHY

---

**首先需要知道 交互式shell和非交互式shell、登录shell和非登录shell是有区别的**

- 在登录shell里，环境信息需要读取`/etc/profile`和`~ /.bash_profile,` `~/.bash_login`, and `~/.profile`按顺序最先的一个，并执行其中的命令。除非被 `--noprofile`选项禁止了；
- 在非登录shell里，**环境信息只读取 `/etc/bash.bashrc`和`~/.bashrc`**
  手工执行是属于登陆shell，脚本执行数据非登陆shell，而我的linux环境配置中只对`/etc/profile`进行了jdk1.7等环境的配置，所以脚本执行`/usr/local/zookeeper/bin/zkServer.sh start `启动zookeeper失败了

<br>



# Answer

---

**解决方法（下面3个方法任选1）：**

1. 脚本代码中添加`source /etc/profile`;

   `ssh crxy$i "source /etc/profile;/usr/local/zookeeper/bin/zkServer.sh start" `

2. 把profile的配置信息echo到.bashrc中 

    `echo 'source /etc/profile' >> ~/.bashrc`

3. 在`/zookeeper/bin/zkEnv.sh`的中开始位置添加 `export JAVA_HOME=/usr/local/jdk1.7.0_45`（就像hadoop中对hadoop-env.sh的配置一样）
   