---
title: JAVA WAB系列--谈谈servlet
date: 2019-07-14 21:09:23
tags:
- Java
- Web
- Servlet
categories:
- Java
- Web
top:
mathjax:
---



{% cq %}

servlet生命周期➕get方式和post方式的区别➕forward和redirect的区别➕Session和cookie有什么区别➕jsp和servlet的区别、共同点、各自应用的范围➕属性作用域

{% endcq %}

<!-- more -->

# servlet生命周期

---

## **Servlet生命周期可分为5个步骤:**

1. **加载Servlet**。当Tomcat第一次访问Servlet的时候，**Tomcat会负责创建Servlet的实例**
2. **初始化**。当Servlet被实例化后，Tomcat会**调用init()方法初始化这个对象**
3. **处理服务**。当浏览器**访问Servlet**的时候，Servlet **会调用service()方法处理请求**
4. **销毁**。当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动调用destroy()方法，**让该实例释放掉所占的资源**。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁
5. **卸载**。当Servlet调用完destroy()方法后，等待垃圾回收。如果**有需要再次使用这个Servlet，会重新调用init()方法进行初始化操作**。

## **总结**

**只要访问Servlet，service()就会被调用。init()只有第一次访问Servlet的时候才会被调用。destroy()只有在Tomcat关闭的时候才会被调用。**

<br>

# get方式和post方式的区别

---

## 数据携带

- GET方式：在URL地址后附带的参数是有限制的，其数据容量通常不能超过1K。
- POST方式：可以在请求的实体内容中向服务器发送数据，传送的数据量无限制。

## 请求参数的位置

- GET方式：请求参数放在URL地址后面，以?的方式来进行拼接
- POST方式:请求参数放在HTTP请求包中

## 用途

- GET方式一般用来获取数据
- POST方式一般用来提交数据
  - 原因:
    - 首先是因为GET方式携带的数据量比较小，无法带过去很大的数量
    - POST方式提交的参数后台更加容易解析(使用POST方式提交的中文数据，后台也更加容易解决)
    - GET方式比POST方式要快

<br>

# forward和redirect的区别

---

## **实际发生位置不同，地址栏不同**

- 转发是发生在服务器的
**转发是由服务器进行跳转的**，细心的朋友会发现，在转发的时候，**浏览器的地址栏是没有发生变化的**，在我访问Servlet111的时候，即使跳转到了Servlet222的页面，浏览器的地址还是Servlet111的。也就是说**浏览器是不知道该跳转的动作，转发是对浏览器透明的**。通过上面的转发时序图我们也可以发现，**实现转发只是一次的http请求**，**一次转发中request和response对象都是同一个**。这也解释了，为什么可以使用**request作为域对象进行Servlet之间的通讯。**

- 重定向是发生在浏览器的
**重定向是由浏览器进行跳转的**，进行重定向跳转的时候，**浏览器的地址会发生变化的**。曾经介绍过：实现重定向的原理是由response的状态码和Location头组合而实现的。**这是由浏览器进行的页面跳转**实现重定向**会发出两个http请求**，**request域对象是无效的，因为它不是同一个request对象**

## **用法不同:**

- 很多人都搞不清楚转发和重定向的时候，**资源地址究竟怎么写**。有的时候要把应用名写上，有的时候不用把应用名写上。很容易把人搞晕。记住一个原则： **给服务器用的直接从资源名开始写，给浏览器用的要把应用名写上**
    - request.getRequestDispatcher("/资源名 URI").forward(request,response)
      - **转发时"/"代表的是本应用程序的根目录【zhongfucheng】**
    - response.send("/web应用/资源名 URI");
      - **重定向时"/"代表的是webapps目录**

## **能够去往的URL的范围不一样:**

  - 转发是服务器跳转只能去往当前web应用的资源
  - 重定向是服务器跳转，可以去往任何的资源

## **传递数据的类型不同**

  - 转发的request对象可以传递各种类型的数据，包括对象
  - 重定向只能传递字符串

## **跳转的时间不同**

  - 转发时：执行到跳转语句时就会立刻跳转
  - **重定向：整个页面执行完之后才执行跳转**

## 那么转发(forward)和重定向(redirect)使用哪一个？

- 根据上面说明了转发和重定向的区别也可以很容易概括出来**。转发是带着转发前的请求的参数的。重定向是新的请求**。

**典型的应用场景：**

1. 转发: 访问 Servlet 处理业务逻辑，然后 forward 到 jsp 显示处理结果，浏览器里 URL 不变
2. 重定向: 提交表单，处理成功后 redirect 到另一个 jsp，防止表单重复提交，浏览器里 URL 变了

<br>

# Session和cookie有什么区别？
---

## **从存储方式上比较**
  - Cookie只能存储字符串，如果要存储非ASCII字符串还要对其编码。
  - Session可以存储任何类型的数据，可以把Session看成是一个容器
## **从隐私安全上比较**
  - **Cookie存储在浏览器中，对客户端是可见的**。信息容易泄露出去。如果使用Cookie，最好将Cookie加密
  - **Session存储在服务器上，对客户端是透明的**。不存在敏感信息泄露问题。
## **从有效期上比较**
  - Cookie保存在硬盘中，只需要设置maxAge属性为比较大的正整数，即使关闭浏览器，Cookie还是存在的
  - **Session的保存在服务器中，设置maxInactiveInterval属性值来确定Session的有效期。并且Session依赖于名为JSESSIONID的Cookie，该Cookie默认的maxAge属性为-1。如果关闭了浏览器，该Session虽然没有从服务器中消亡，但也就失效了。**
## **从对服务器的负担比较**
  - Session是保存在服务器的，每个用户都会产生一个Session，如果是并发访问的用户非常多，是不能使用Session的，Session会消耗大量的内存。
  - Cookie是保存在客户端的。不占用服务器的资源。像baidu、Sina这样的大型网站，一般都是使用Cookie来进行会话跟踪。
## **从浏览器的支持上比较**
  - 如果浏览器禁用了Cookie，那么Cookie是无用的了！
  - 如果浏览器禁用了Cookie，Session可以通过URL地址重写来进行会话跟踪。
## **从跨域名上比较**
  - Cookie可以设置domain属性来实现跨域名
  - Session只在当前的域名内有效，不可夸域名

<br>

# jsp和servlet的区别、共同点、各自应用的范围

---

- JSP是Servlet技术的扩展，**本质上就是Servlet的简易方式**。JSP编译后是“类servlet”。
- Servlet和JSP最主要的不同点在于：Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML里分离开来。**而JSP的情况是Java和HTML可以组合成一个扩展名为.jsp的文件。**
- **JSP侧重于视图，Servlet主要用于控制逻辑。**

<br>

# 属性作用域范围

---
## 属性作用域
- **page【只在一个页面中保存属性，跳转页面无效】**
- **requet【只在一次请求中保存属性，服务器跳转有效，浏览器跳转无效】**
- **session【在一个会话范围中保存属性，无论何种跳转均有效，关闭浏览器后无效】**
- **application/servletcontext【在整个服务器中保存，所有用户都可以使用】**

## 应用场景

1. request：如果客户向服务器发请求，产生的数据，**用户看完就没用了，**像这样的数据就存在request域,像新闻数据，属于用户看完就没用的
2. session：如果客户向服务器发请求，产生的数据，**用户用完了等一会儿还有用**，像这样的数据就存在session域中，像购物数据，用户需要看到自己购物信息，并且等一会儿，还要用这个购物数据结帐
3. application/servletContext：如果客户向服务器发请求，产生的数据，**用户用完了，还要给其它用户用**，像这样的数据就存在servletContext域中，像聊天数据