---
title: JavaWeb
date: 2019-07-14 15:50:03
description: 
tags: 
- Java
- Web
- servlet
- session
- cookie
- forward
- redirect
- 面试
categories: 
- Java
- Web
top:
---

{% cq %}forward和redirect的区别;➕get方式和post方式有何区别;➕什么是cookie?➕session和cookie的区别?;➕Servelet调用流程;➕servletvcontext对象;➕servlet之间进行通信{% endcq %}

<!-- more -->

# **forward和redirect的区别**

------

## **实际发生位置不同,地址栏不同**

- **转发是发生在服务器上**  
  **转发是由服务器进行跳转的**，细心的朋友会发现，在转发的时候，**浏览器的地址栏是没有发生变化的**，在我访问Servlet111的时候，即使跳转到了Servlet222的页面，浏览器的地址还是Servlet111的。也就是说**浏览器是不知道该跳转的动作，转发是对浏览器透明的**。通过上面的转发时序图我们也可以发现，**实现转发只是一次的http请求**，**一次转发中request和response对象都是同一个**。这也解释了，为什么可以使用**request作为域对象进行Servlet之间的通讯。**
- **重定向是发生在浏览器的**  
  **重定向是由浏览器进行跳转的**，进行重定向跳转的时候，**浏览器的地址会发生变化的**。曾经介绍过：实现重定向的原理是由response的状态码和Location头组合而实现的。**这是由浏览器进行的页面跳转**实现重定向**会发出两个http请求**，**request域对象是无效的，因为它不是同一个request对象**

## **用法不同:**

很多人都搞不清楚转发和重定向的时候，**资源地址究竟怎么写**。有的时候要把应用名写上，有的时候不用把应用名写上。很容易把人搞晕。记住一个原则： **给服务器用的直接从资源名开始写，给浏览器用的要把应用名写上**  

```js
request.getRequestDispatcher("/资源名 URI").forward(request,response)
```

> 转发时"/"代表的是本应用程序的根目录【zhongfucheng  

```js
response.send("/web应用/资源名 URI");
```

> 重定向时"/"代表的是webapps目录  

## **能够去往的URL的范围不一样:**

- **转发是服务器跳转只能去往当前web应用的资源**
- **重定向是服务器跳转，可以去往任何的资源**

## **传递数据的类型不同**

- **转发的request对象可以传递各种类型的数据，包括对象**
- **重定向只能传递字符串**

## **跳转的时间不同**

- **转发时：执行到跳转语句时就会立刻跳转**
- **重定向：整个页面执行完之后才执行跳转**

## **那么转发(forward)和重定向(redirect)使用哪一个？**

根据上面说明了转发和重定向的区别也可以很容易概括出来**。转发是带着转发前的请求的参数的。重定向是新的请求**。

## **典型的应用场景：**  

- 转发: 访问 Servlet 处理业务逻辑，然后 forward 到 jsp 显示处理结果，浏览器里 URL 不变
- 重定向: 提交表单，处理成功后 redirect 到另一个 jsp，防止表单重复提交，浏览器里 URL 变了

<br>

# **get方式和post方式有何区别**

------

## **数据携带上**

- get方式:在URL地址后附带的参数是有限制的,其数据流量通常不超过1kb
- post方式:可以在请求的实体内容中向服务器发送数据,传送的数据量不限制

## **请求参数的位置上**

- get方式:请求参数放在URL地址后边,以?的形式来进行拼接
- post方式:请求参数放在HTTP请求包中

## **用途上**

- get方式:一般用来获取数据
- post方式:一般用来提交数据

> **原因:**  
> 首先是因为get方式携带的数据量比较小,无法携带很大的数据量  
> post方式提交的参数后台更加容易解析(使用post方式提交的中文数据,后台更加容易解决)  
> get方式比post方式要快  

<br>

# **什么是cookie?session和cookie的区别?**

------

## **什么是cookie**

- 网页之间的交互是通过HTTP的协议传输数据的,而HTTP协议是无状态的协议.一旦数据提交上去后,浏览器与服务器的连接就会关闭,再次交互需要重新建立新的连接
- 服务器无法确认用户端信息.所以给每个用户都发一个通行证,无论谁访问的时候都需要携带通行证,这样的话服务器就可以确认用户信息

## **session和cookie的区别:**

### **从存储方式上比较**

- Cookie只能存储字符串，如果要存储非ASCII字符串还要对其编码。
- Session可以存储任何类型的数据，可以把Session看成是一个容器

### **从隐私安全上比较**

- **Cookie存储在浏览器中，对客户端是可见的**。信息容易泄露出去。如果使用Cookie，最好将Cookie加密
- **Session存储在服务器上，对客户端是透明的**。不存在敏感信息泄露问题。

### **从有效期上比较**

- Cookie保存在硬盘中，只需要设置maxAge属性为比较大的正整数，即使关闭浏览器，Cookie还是存在的
- **Session的保存在服务器中，设置maxInactiveInterval属性值来确定Session的有效期。并且Session依赖于名为JSESSIONID的Cookie，该Cookie默认的maxAge属性为-1。如果关闭了浏览器，该Session虽然没有从服务器中消亡，但也就失效了。**

### **从对服务器的负担比较**

- Session是保存在服务器的，每个用户都会产生一个Session，如果是并发访问的用户非常多，是不能使用Session的，Session会消耗大量的内存。
- Cookie是保存在客户端的。不占用服务器的资源。像baidu、Sina这样的大型网站，一般都是使用Cookie来进行会话跟踪。

### **从浏览器的支持上比较**

- 如果浏览器禁用了Cookie，那么Cookie是无用的了！
- 如果浏览器禁用了Cookie，Session可以通过URL地址重写来进行会话跟踪。

### **从跨域名上比较**

- Cookie可以设置domain属性来实现跨域名
- Session只在当前的域名内有效，不可夸域名

<br>

# **Servelet调用流程**

------

1. 连接web服务器(浏览器)
2. 发送HTTP请求(浏览器)
3. 解析出客户端想访问的主机(web服务器)
4. 解析出客户端想访问的web资源(web服务器)
5. 如果发现servlet是第一次访问,服务器就会第一次加载servlet.并且初始化servlet对象(web服务器)
6. 调用servlet类的init方法(servlet)
7. service方法执行,把数据写入response中(servlet)
8. web服务器把response数据传回浏览器(Web服务器)

<br>

# **servletvcontext对象**

------

## **概念**

当tomcat启动的时候,就会创建一个servletcontext对象.他代表着当前web站点  

## **servletcontext怎么用**

- servletcontext代表着当前web页面,那么所有的servlet对象都共享着同一个servletcontext对象,那么所有的servlet可以通过servletContext进行通信
- servletconfig获取的是单个servlet的参数信息,servletcontext可以获取的是配置整个web站点的参数信息
- 利用servletcontext读取web站点的资源文件
- 实现servlet的转发(多数使用request的转发)

<br>

# **servlet之间进行通信**

------

servletcontext对象可以被称为域对象(可以理解成为容器,类似于map集合)  

实现servlet之间的通信就要用到servletcontext的setattribute(string name,object obj)方法,第一个参数是关键字,第二个参数是你要存储的对象

## **demo1:**

```java
        //获取到ServletContext对象
        ServletContext servletContext = this.getServletContext();

        String value = "mark";

        //MyName作为关键字，value作为值存进   域对象【类型于Map集合】
        servletContext.setAttribute("MyName", value);
```



## **demo2:**

```java
        //获取ServletContext对象
        ServletContext servletContext = this.getServletContext();

        //通过关键字获取存储在域对象的值
        String value = (String) servletContext.getAttribute("MyName");

        System.out.println(value);
```





