---
title: SQL中的where  ！= 歧义
date: 2021-10-09 10:34:11
updated: 2021-10-09 10:34:11
comments: true
---
null在sql是一种不确定的值，可以认为系统也不知道这个值是什么，所以不管用=还是!=都是会把null过滤掉的。另一方面来说，''（空字符串）可以认为是一个确定的值，值就是空，而null本身就是没有的。

涉及到null的要使用is null或is not null