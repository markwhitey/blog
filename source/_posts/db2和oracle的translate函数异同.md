---
title: DB2和Oracle的TRANSLATE函数异同
date: 2020-10-10 13:35:47
updated: 2020-10-10 13:35:47
tags:
  - DB2
comments: true
---
{% cq %}

TRANSLATE 在不同数据库中含义不同

{% endcq %}

<!-- more -->

<br>


oracle里：
             TRANSLATE(str1,str2,str3)

              用str3里对应的字段，逐一替换str1中的str2字段。

            例  

                        select  TRANSLATE('12345bcdm','1234567abcdem','9999999xxxxxx') from dual ;

                         结果显示 ： 99999xxxx

                       select  TRANSLATE( '2011-01-01',  ' 0123456789-','')  from dual

                         结果显示 ：null（因oracle里没有空串）

              即缺省替换为 ' '（空格）

db2 里：

            TRANSLATE(str1,str2,str3)

             于oracle里刚好相反，以str2替换str3

             例

                      select  translate('012-38(3)','***','-()')  from sysibm.SYSDUMMY1

                      结果为：012*38*3*

                     select  TRANSLATE( '2011-01-01', '', ' 0123456789-' )  from sysibm.SYSDUMMY1  

                      结果为   ''   即空串（不是null，上方oralce里第二个例子结果为null，因oracle里没有空串）

                  同样缺省替换为' '（空格）