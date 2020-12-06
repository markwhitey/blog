---
title: "impala 系列: 时间函数"
date: 2020-11-30 10:49:34
updated: 2020-11-30 10:49:34
tags:
  - Impala
categories:
  - Impala
comments: true
---
{% cq %}
关于时间函数
{% endcq %}

<!-- more -->

<br>


-- 当前时间戳
now()
current_timestamp()

-- 当前时间戳相对于 linux epoch 的秒数
unix_timestamp() , 不带参数, 则返回 '1970-01-01 00:00:00' UTC 到现在的秒数

-- 转换到相对于 linux epoch 的秒数
unix_timestamp(now()+ interval 3 days), 如果传入 timestamp 参数, 返回该时间戳相对于 linux epoch 的秒数
unix_timestamp(string datetime, string format), 还支持传入时间字符串, 返回值还是相对于 linux epoch 的秒数

-- 将秒数转换到字符串
from_unixtime(int, 'yyyy/MM/dd HH:mm'), 将指定的时间戳, 格式化为字符串. 时间戳参数应该是秒数格式, 所以该参数需要用 unix_timestamp() 包一下.
注意月份和分钟对应的格式字符串, 常用的格式有 "yyyy-MM-dd HH:mm:ss.SSSSSS", "dd/MM/yyyy HH:mm:ss.SSSSSS", "MMM dd, yyyy HH.mm.ss (SSSSSS)"

-- 将时间戳转换为日期字符串
to_date(timestamp) , 将指定时间戳转换为日期字符串, 日期格式为 yyyy--MM-dd .

-- 将秒数转换成时间戳
to_timestamp(bigint unixtime)

-- 将字符串转换成时间戳
to_timestamp(string date, string pattern)

说明: impala 没有直接将时间戳转换为字符串的函数, 所以经常的写法是: from_unixtime(unix_timestamp( t1),'yyyyMMdd HH:mm')

-- 时间戳取整
Impala 2.11 之前的取整当前时间的写法:
select trunc(now(), 'YEAR') -- 取整到年份, 得到当年 1 月 1 日 0 点 0 分
select trunc(now(), 'MONTH') -- 取整到月份, 得到当月 1 日 0 点 0 分
select trunc(now(), 'DD') -- 取整到日期, 得到当天 0 点 0 分
select trunc(now(), 'DAY') -- 取整到星期, 得到本星期第一天的 0 点 0 分
select trunc(now(), 'HH24') -- 取整到小时, 得到当前小时的 0 分
select trunc(now(), 'MI') -- 取整到分钟, 得到当前分钟 0 秒


Impala 2.11 之后增加了 date_trunc() 函数, 下面是几个取整的写法:
date_trunc('year',now())
date_trunc('month',now())
date_trunc('week',now())
date_trunc('day',now())
date_trunc('hour',now())
date_trunc('minute',now())
date_trunc() 的语法和 date_part() 类似, 下面是完整的时间 part 列表:
microseconds
milliseconds
second
minute
hour
day
week
month
year
decade
century
millennium


-- 时间戳提取
date_part('year', now())
extract(now(), 'year')
extract(year from now())


-- 两个时间戳比较
datediff(timestamp enddate, timestamp startdate) , 相差多少天, 精度是天
timestamp_cmp(now() + interval 70 minutes, now()), 比较两个时间戳的大小, 本例的结果为 1
impala 没有好用的 timestamp_diff() 函数, 比如我们想要知道两个时间相差多少个小时, 不能直接求出, 下面是一个简单的步骤:
1. 先算出一个小时对应的秒数是多少
2. 将两个时间都转成秒数, 然后做差, 然后除以一个小时的秒数.


-- 时间加减
时间戳可以直接加减 interval n days/months/years/hours/minutes .

也可以使用下面的函数:
years_add(timestamp t, int n)
years_sub(timestamp t, int n)
months_add(timestamp t, int n)
months_sub(timestamp t, int n)
days_add(timestamp t, int n)
days_sub(timestamp t, int n)
hours_add(timestamp t, int n)
hours_sub(timestamp t, int n)
minutes_add(timestamp t, int n)
minutes_sub(timestamp t, int n)

也可以用下面两个通用的函数:
date_add(timestamp startdate, int days)
date_add(timestamp startdate, interval_expression)
date_sub(timestamp startdate, int days)
date_sub(timestamp startdate, interval_expression)


-- 月份相关的
last_day(timestamp t)
months_between(timestamp newer, timestamp older)