---
title: listener与filter
date: 2019-07-14 18:56:02
tags:
- Java
- listener
- filter
- servlet
- 面试
categories: 
- Java
- Web
top:
mathjax:
---



{% cq %}

Filter是一个可重复使用的代码片段，可以用来转换HTTP请求、响应和头信息。Filter不像Servlet，它不能产生一个请求或者响应，它只是修改对某一资源的请求，或者修改从某一资源的响应。

Listener是监听器，通过Listener可以监听Web服务器中某一个执行动作，并根据其要求作出相应的响应。通俗的来说就是在Application，Session，Request三个对象创建、消亡或者往其中添加、修改、删除属性时自动执行代码功能的组件。

{% endcq %}

<!-- more -->

# **监听器有哪些作用和用法**

------

Java Web开发中的监听器（listener）就是application、session、request三个对象创建、销毁或者往其中添加修改删除属性时自动执行代码的功能组件，如下所示：

- ServletContextListener：对Servlet上下文的创建和销毁进行监听。
- ServletContextAttributeListener：监听Servlet上下文属性的添加、删除和替换。
- HttpSessionListener：对Session的创建和销毁进行监听。

> 补 充：session的销毁有两种情况：  
> session超时（可以在web.xml中通过`<session- config>/<session-timeout>`标签配置超时时间）；  
> 通过调用session对象的invalidate()方 法使session失效。

- HttpSessionAttributeListener：对Session对象中属性的添加、删除和替换进行监听。
- ServletRequestListener：对请求对象的初始化和销毁进行监听。
- ServletRequestAttributeListener：对请求对象属性的添加、删除和替换进行监听。

常见的监听器用途主要包括：**网站在线人数技术、监听用户的行为(管理员踢人)**。

<br>

# **监听器的用法**

------

**ServletContextListener:监听ServletContext对象的创建和销毁**

- 方法：
  1. void contextDestroyed(ServletContextEvent sce) ：ServletContext对象被销毁之前会调用该方法
  2. void contextInitialized(ServletContextEvent sce) ：ServletContext对象创建后会调用该方法
- 步骤：
  1. 定义一个类，实现ServletContextListener接口
  2. 复写方法
  3. 配置
     1. web.xml
     2. 注解配置@WebListener

```java
<listener>
<listener-class>
cn.listener.ContextLoaderListener
</listener-class
</listener>
```


<br>

# **过滤器的作用**

------

Java Web开发中的过滤器（filter）是从Servlet 2.3规范开始增加的功能，并在Servlet 2.4规范中得到增强。对Web应用来说，**过滤器是一个驻留在服务器端的Web组件**，它可以截取客户端和服务器之间的请求与响应信息，并对这些信息进行过 滤。当Web容器接受到一个对资源的请求时，它将判断是否有过滤器与这个资源相关联。如果有，那么容器将把请求交给过滤器进行处理。在过滤器中，**你可以改 变请求的内容，或者重新设置请求的报头信息，然后再将请求发送给目标资源。当目标资源对请求作出响应时候，容器同样会将响应先转发给过滤器，再过滤器中， 你可以对响应的内容进行转换，然后再将响应发送到客户端。**

常见的过滤器用途主要包括：**对用户请求进行统一认证、对用户的访问请求进行记录和审核、对用户发送的数据进行过滤或替换、转换图象格式、对响应内容进行压缩以减少传输量、对请求或响应进行加解密处理、触发资源访问事件、对XML的输出应用XSLT等**。

和过滤器相关的接口主要有：Filter、FilterConfig、FilterChain

<br>

# **过滤器使用方法**

------

1. 步骤：
   1. 定义一个类，实现接口Filter
   2. 复写方法
   3. 配置拦截路径
      1. web.xml
      2. 注解
2. 代码：

```java
			@WebFilter("/*")//访问所有资源之前，都会执行该过滤器
			public class FilterDemo1 implements Filter {
			    @Override
			    public void init(FilterConfig filterConfig) throws ServletException {
			
			    }
			
			    @Override
			    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
			        System.out.println("filterDemo1被执行了....");

			
			        //放行
			        filterChain.doFilter(servletRequest,servletResponse);
			
			    }
			
			    @Override
			    public void destroy() {
			
			    }
			}
```

# **过滤器细节**

1. **web.xml配置**	

```java
		<filter>
	        <filter-name>demo1</filter-name>
	        <filter-class>web.filter.FilterDemo1</filter-class>
	    </filter>
	    <filter-mapping>
	        <filter-name>demo1</filter-name>
			<!-- 拦截路径 -->
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
```

2. **过滤器执行流程**
   1. 执行过滤器
   2. 执行放行后的资源
   3. 回来执行过滤器放行代码下边的代码
3. **过滤器生命周期方法**
   1. init:在服务器启动后，会创建Filter对象，然后调用init方法。只执行一次。用于加载资源
   2. doFilter:每一次请求被拦截资源时，会执行。执行多次
   3. destroy:在服务器关闭后，Filter对象被销毁。如果服务器是正常关闭，则会执行destroy方法。只执行一次。用于释放资源
4. **过滤器配置详解**
   - **拦截路径配置**：
     1. 具体资源路径： /index.jsp   只有访问index.jsp资源时，过滤器才会被执行
     2. 拦截目录： /user/*	访问/user下的所有资源时，过滤器都会被执行
     3. 后缀名拦截： *.jsp		访问所有后缀名为jsp资源时，过滤器都会被执行
     4. 拦截所有资源：/**		访问所有资源时，过滤器都会被执行
   - **拦截方式配置：资源被访问的方式**
     - 注解配置：
       - 设置dispatcherTypes属性
         1. REQUEST：默认值。浏览器直接请求资源
         2. FORWARD：转发访问资源
         3. INCLUDE：包含访问资源
         4. ERROR：错误跳转资源
         5. ASYNC：异步访问资源
     - web.xml配置
       - 设置<dispatcher></dispatcher>标签即可
5. **过滤器链(配置多个过滤器)**
   - 执行顺序：如果有两个过滤器：过滤器1和过滤器2
     1. 过滤器1
     2. 过滤器2
     3. 资源执行
     4. 过滤器2
     5. 过滤器1 
   - 过滤器先后顺序问题：
     1. 注解配置：按照类名的字符串比较规则比较，值小的先执行
        - 如： AFilter 和 BFilter，AFilter就先执行了。
     2. web.xml配置： <filter-mapping>谁定义在上边，谁先执行

