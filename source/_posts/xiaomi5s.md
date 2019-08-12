---
title: XIAOMI-5S国际版刷机复活计划
mathjax: false
date: 2019-08-11 19:08:44
tags:
- Fun
categories:
- Life
top:
photo:
---



{% cq %}

**小米5s魔改6g+国际版刷机过程**

{% endcq %}

<!-- more -->

<br>

# 手机情况

---

> 小米5s作为小米系列的旗舰级手机,实在是有些不太对得起"旗舰"的称号,首发的时候只有3g的运存,但是却有着骁龙821的处理器,所以,这台家里不用的手机我想怎么才能折腾一下呢?

## 万能的淘宝

首先想到的就是万能的淘宝,果然一搜就有这样的魔改业务,加上手机之前摔过以后出现屏幕失灵的情况,索性直接体检一下,最后手机修了芯片和魔改了一下从3g到6g,一共花了208,效果还是不错的,不过还是推荐大家从手机店里修,毕竟可以有问题了可以直接去找

## 小米国际版
普通国际版的MIUI虽然没有国内的MIUI广告多，但是仍有AnalyticsCore、MSA-Global等广告后台。

真正纯净的国际版MIUI是得到欧盟官方分销商ABC Data认可的[MIUI by xiaomi.eu](https://link.zhihu.com/?target=https%3A//xiaomi.eu/)。

{% tabs 手机界面, 1%}
<!-- tab 桌面及负一屏-->

{% gp 3-1 %}
<img src="https://i.loli.net/2019/08/11/BzPmao6HvK1Sq5A.jpg" >
<img src="https://i.loli.net/2019/08/11/DgucyPYIX5dQSEL.jpg">
<img src="https://i.loli.net/2019/08/11/phJxAeRNlMPI2qK.jpg">
{% endgp %}

<!-- endtab -->
<!-- tab 手机全部参数-->
<img src="https://i.loli.net/2019/08/11/9KJXsjoOcuv3MAq.jpg" width="45%">
<!-- endtab -->
<!-- tab 主题界面-->
**国际版的主题都是免费的**
<img src="https://i.loli.net/2019/08/11/aEOg7bFSBNUpk6u.jpg" width="45%">
<!-- endtab -->
<!-- tab 小米视频和小米音乐-->
**其中的小米音乐、小米视频等都是做过去广告的特殊处理，打开后是个完全纯净的播放器。整个系统的干净程度可与ios系统相比。**
{% gp 2-2 %}
<img src="https://i.loli.net/2019/08/11/OskfhxPlXY7U6nz.jpg" >
<img src="https://i.loli.net/2019/08/11/QNXKqBD8Lpoug7C.jpg" >
{% endgp %}
<!-- endtab -->
{% endtabs %}

<br>

# 刷机教程

---

## 第一步 解锁(必须)

解锁前要先备份数据，因为会恢复到出厂设置。[下载解锁工具](https://www.miui.com/unlock/index.html)安装到电脑上，根据提示解锁。成功后开机会显示 unlocked。(不知道为什么我需要挂代理才能访问)

## 第二步  刷 recovery 

- 我们可以在[小米论坛](http://bbs.xiaomi.cn/f-391)上找到适合自己手机型号的twrp专版
- 选择 TWRP-XXX-CN-wzsx150-fastboot.7z 下载后解压，手机连接电脑，双击运行一键刷入 recovery 工具. bat 文件，按照提示操作，最终通过 fastboot 模式将 rec 刷入手机。
- 刷入成功后进入 TWRP
  - 清除 - 格式化 Data 分区 - 执行；重启 rec；
  - 清除 - 高级清除选项 - 勾选 Dalvik/ART  Cache  DataCache  System （四清）执行；然后重启 rec。

## 第三步  下载国际版ROM并传输卡刷包到手机

- [xiaomi.eu](https://sourceforge.net/projects/xiaomi-eu-multilang-miui-roms/files/xiaomi.eu/MIUI-STABLE-RELEASES/MIUIv10/#)访问该网址并且下载自己适合手机的rom的zip文件(不要解压，复制到手机根目录)
- TWRP 界面点击安装 - 选择你的刷机包 rom - 执行
- 结束重启手机，第一次重启会有点慢。**激活时国内用户要跳过[谷歌](https://pinpai.smzdm.com/2047/)验证，爬墙后再验证**。(**关闭wifi以及拔掉sim卡!!**)

## 第四步 验证谷歌play

> Google 最近开始限制其全家桶的使用，未经过认证的设备会在商店里提示设备未认证，但实际上第三方ROM 用户可以自行到Google 网站上认证。

- 使用这款名为[Device ID](https://play.google.com/store/apps/details?id=com.evozi.deviceid) 的APP 复制你的GSF ID

  <img src="https://i.loli.net/2019/08/11/VhpXq5eFUIdO3w6.png" width="45%">

- 打开Google官方设备[认证页面](https://www.google.com/android/uncertified/)，输入刚刚所获得GSF ID，单击注册即可

  注册完成后，在页面下方会显示已注册的安卓设备ID

- 然后就可以在google商店里设置的最下方看到设备认证

  <img src="https://i.loli.net/2019/08/11/T3DEAxsHKjtRoCX.jpg" width="45%">