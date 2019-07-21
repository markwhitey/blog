---
title: Mybatis
date: 2019-07-21T16:48:37.000Z
tags:
  - Mybatis
  - 面试
categories:
  - Java框架
  - Mybatis
  - 基础
---
{% cq %}

Mybatis 是一个半 ORM（对象关系映射）框架，它内部封装了 JDBC，开发时只需要关注 SQL 语句本身，不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。程序员直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高。

{% endcq %}

<!-- more -->

# 什么是mybatis?

- - -

* Mybatis 是一个半 ORM（对象关系映射）框架，它内部封装了 JDBC，开发时只需要关注 SQL 语句本身，不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。程序员直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高.
* MyBatis 可以使用 XML 或注解来配置和映射原生信息，将 POJO 映射成数据库中的记录，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集.
* 通过 xml 文件或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中 sql 的动态参数进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql 并将结果映射为 java 对象并返回。（从执行 sql 到返回 result 的过程）。

<br>

# mybatis的优缺点及适用场合?

- - -

## 优点:

1. 基于 SQL 语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL 写在 XML 里，解除 sql 与程序代码的耦合，便于统一管理；提供 XML 标签，支持编写动态 SQL 语句，并可重用。  
2. 与 JDBC 相比，减少了 50% 以上的代码量，消除了 JDBC 大量冗余的代码，不需要手动开关连接；  
3. 很好的与各种数据库兼容（因为 MyBatis 使用 JDBC 来连接数据库，所以只要 JDBC 支持的数据库 MyBatis 都支持）。  
4. 能够与 Spring 很好的集成；  
5. 提供映射标签，支持对象与数据库的 ORM 字段关系映射；提供对象关系映射标签，支持对象关系组件维护。

## 缺点:

1. SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求。  
2. SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

## 适用场合:

1. MyBatis 专注于 SQL 本身，是一个足够灵活的 DAO 层解决方案。  
2. 对性能的要求很高，或者需求变化较多的项目，如互联网项目，MyBatis 将是不错的选择。

<br>

# MyBatis 与 Hibernate 有哪些不同？

- - -

1. Mybatis 和 hibernate 不同，它不完全是一个 ORM 框架，因为 MyBatis 需要程序员自己编写 Sql 语句。  
2. Mybatis 直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需求变化要求迅速输出成果。但是灵活的前提是 mybatis 无法做到数据库无关性，如果需要实现支持多种数据库的软件，则需要自定义多套 sql 映射文件，工作量大。  
3. Hibernate 对象 / 关系映射能力强，数据库无关性好，对于关系模型要求高的软件，如果用 hibernate 开发可以节省很多代码，提高效率。

<br>

# \#{} 和 ${} 的区别是什么？

- - -

* \#{} 是预编译处理，${} 是字符串替换。    
* Mybatis 在处理 #{} 时，会将 sql 中的 #{} 替换为? 号，调用 PreparedStatement 的 set 方法来赋值；  
* Mybatis 在处理 ${} 时，就是把 ${} 替换成变量的值。  
* 使用 #{} 可以有效的防止 SQL 注入，提高系统安全性。

<br>

# 为什么说 Mybatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？

- - -

* Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。
* Mybatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。

<br>

# MyBatis 实现一对一有几种方式? 具体怎么操作的？

- - -

* **联合查询**:几个表联合查询, 只查询一次, 通过在 resultMap 里面配置 association 节点配置一对一的类就可以完成；  
* **嵌套查询**:先查一个表，根据这个表里面的结果的 外键 id，去再另外一个表里面查询数据, 也是通过 association 配置，但另外一个表的查询通过 select 属性配置。

<br>

# Mybatis 的缓存

- - -

## 概念

* 一级缓存：一级缓存是 SqlSession（会话）级别的缓存。在操作数据库时需要构造 sqlSession 对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的 sqlSession 之间的缓存数据区域（HashMap）是互相不影响的。
* 二级缓存：二级缓存是 mapper 级别的缓存，多个 SqlSession 去操作同一个 Mapper 的 sql 语句，多个 SqlSession 可以共用二级缓存，二级缓存是跨 SqlSession 的。（二级缓存的原理和一级缓存原理一样，第一次查询，会将数据放入缓存中，然后第二次查询则会直接去缓存中取。但是一级缓存是基于 sqlSession 的，而 二级缓存是基于 mapper 文件的 namespace 的，也就是说多个 sqlSession 可以共享一个 mapper 中的二级缓存区域，并且如果两个 mapper 的 namespace 相同，即使是两个 mapper，那么这两个 mapper 中执行 sql 查询到的数据也将存在相同的二级缓存区域中）  

它们的关系如图：

![2476857336-5b4b312db5aa1_articlex.png](https://i.loli.net/2019/07/21/5d344937d093076066.png)

## 一级缓存的使用

我们在一个 sqlSession 中，对 User 表根据 id 进行两次查询，查看他们发出 sql 语句的情况：

* 第一次发起查询用户 id 为 1 的用户信息，先去找缓存中是否有 id 为 1 的用户信息，如果没有，从数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。 
* 如果中间 sqlSession 去执行 commit 操作（执行插入、更新、删除），则会清空 SqlSession 中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。
* 第二次发起查询用户 id 为 1 的用户信息，先去找缓存中是否有 id 为 1 的用户信息，缓存中有，直接从缓存中获取用户信息。

![一级.png](https://i.loli.net/2019/07/21/5d344937bc16f28263.png)

## 二级缓存的使用

* sqlSession1 去查询用户信息，查询到用户信息会将查询数据存储到二级缓存中。
* 如果 SqlSession3 去执行相同 mapper 映射下 sql，执行 commit 提交，将会清空该 mapper 映射下的二级缓存区域的数
* 如果 SqlSession3 去执行相同 mapper 映射下 sql，执行 commit 提交，将会清空该 mapper 映射下的二级缓存区域的数  

![二级.png](https://i.loli.net/2019/07/21/5d344937bc39c60418.png)

## 注意事项

* 如果设置了 cacheEnabled=true; 那么 MyBatis 在执行查询的时候，先查看二级缓存（全局缓存）是否有查询的结果，如果有，直接返回缓存的结果；如果没有，再执行真正的查询，把查询的结果放到缓存中，再把结果返回给用户。
* 二级缓存：mybatis 中，每一个 mapper 都可以有一个二级缓存。使用 <cache> 节点配置。
* 二级缓存使用场景：  
  > 对于访问多的查询请求且用户对查询结果实时性要求不高，此时可采用mybatis二级缓存技术降低数据库访问量，提高访问速度，业务场景比如：耗时较高的统计分析sql、电话账单查询sql等。实现方法如下：通过设置刷新间隔时间，由mybatis每隔一段时间自动清空缓存，根据数据变化频率设置缓存刷新间隔flushInterval，比如设置为30分钟、60分钟、24小时等，根据需求而定。
  >  mybatis二级缓存对细粒度的数据级别的缓存实现不好，比如如下需求：对商品信息进行缓存，由于商品信息查询访问量大，但是要求用户每次都能查询最新的商品信息，此时如果使用mybatis的二级缓存就无法实现当一个商品变化时只刷新该商品的缓存信息而不刷新其它商品的信息，因为mybaits的二级缓存区域以mapper为单位划分的，当一个商品信息变化会将所有商品信息的缓存数据全部清空。解决此类问题可能需要在业务层根据需求对数据有针对性缓存。

<br>

# 当实体类中的属性名和表中的字段名不一样 ，怎么办 ？

- - -

1. 通过在查询的 sql 语句中**定义字段名的别名，让字段名的别名和实体类的属性名一致**

```xml
<select id=”selectorder” parametertype=”int” resultetype=”me.gacl.domain.order”> 
       select order_id id, order_no orderno ,order_price price form orders where order_id=#{id}; 
    </select>
```

2. **通过<resultMap>来映射字段名和实体类属性名的一一对应的关系**

```xml
<select id="getOrder" parameterType="int" resultMap="orderresultmap">
        select * from orders where order_id=#{id}
    </select>
   <resultMap type=”me.gacl.domain.order” id=”orderresultmap”> 
        <!–用id属性来映射主键字段–> 
        <id property=”id” column=”order_id”> 
        <!–用result属性来映射非主键字段，property为实体类属性名，column为数据表中的属性–> 
        <result property = “orderno” column =”order_no”/> 
        <result property=”price” column=”order_price” /> 
    </reslutMap>
```

<br>

# 在 mapper 中如何传递多个参数?

- - -

## @param 注解方式



```java
public interface usermapper { 
         user selectuser(@param(“username”) string username, 
         @param(“hashedpassword”) string hashedpassword); 
        }
```

```xml
<select id=”selectuser” resulttype=”user”> 
         select id, username, hashedpassword 
         from some_table 
         where username = #{username} 
         and hashedpassword = #{hashedpassword} 
    </select>
```
