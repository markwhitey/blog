---
title: SSM框架--MVC要点
mathjax: false
date: 2019-08-18 10:04:49
tags:
- Framework
- Web
categories:
- Java Framework
- Spring MVC
top:
photo:
---



{% cq %}

简单记录一下SpringMVC的框架流程以及一些注意事项

{% endcq %}

<!-- more -->

<br>

# SpringMVC工作流程

---

## 总流程图

![](https://i.loli.net/2019/08/18/MlKh14rtJseSgjE.png)

## **请求流程图解**

![](https://i.loli.net/2019/08/18/B46VfordTwMltJC.png)

## **整体判断流程**
![](https://i.loli.net/2019/08/18/8dlZ5biDjGwhAtF.png)

## **详细流程描述**

1. 用户向服务器发送请求，请求被SpringMVC 前端控制器 DispatcherServlet捕获；

2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI）:

   判断请求URI对应的映射	

   - 不存在:
     - 再判断是否配置了mvc:default-servlet-handler：
     - 如果没配置，则控制台报映射查找不到，客户端展示404错误
     - 如果有配置，则执行目标资源（一般为静态资源，如：JS,CSS,HTML）
   - 存在:
     - 执行下面流程

3. 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；

4. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter;

5. 如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法【正向】

6. 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：

   1. HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息
   2. 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等
   3. 数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等
   4. 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中  

7. Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象；

8. 根据返回的ModelAndView（此时会判断是否存在异常：如果存在异常，则执行HandlerExceptionResolver进行异常处理）选择一个适合的ViewResolver（必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet，根据Model和View，来渲染视图

9. 在返回给客户端时需要执行拦截器的AfterCompletion方法【逆向】

10. 将渲染结果返回给客户端.

<br>

# @RequestMapping注解

---

## @RequestMapping 映射请求注解

### @RequestMapping 概念

- SpringMVC使用@RequestMapping注解为控制器指定可以处理哪些 URL 请求

- 在控制器的**类定义及方法定义处**都可标注 @RequestMapping

  - **标记在类上**：提供初步的请求映射信息。相对于  WEB 应用的根目录
  - **标记在方法上**：提供进一步的细分映射信息。相对于标记在类上的 URL

- 若类上未标注 @RequestMapping，则方法处标记的 URL 相对于 WEB 应用的根目录

- **作用**：DispatcherServlet 截获请求后，就通过控制器上 @RequestMapping 提供的映射信息确定请求所对应的处理方法

  
### @ RequestMapping源码参考

```java
package org.springframework.web.bind.annotation;
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
String[] value() default {};
RequestMethod[] method() default {};
String[] params() default {};
String[] headers() default {};
String[] consumes() default {};
String[] produces() default {};
}
```



  ## RequestMapping映射请求占位符PathVariable注解

**带占位符的 URL 是 Spring3.0 新增的功能**，该功能在 SpringMVC 向 **REST** 目标挺进发展过程中具有里程碑的意义

**通过 @PathVariable 可以将 URL 中占位符参数绑定到控制器处理方法的入参中**：

URL 中的 {**xxx**} 占位符可以通过 @PathVariable("**xxx**") 绑定到操作方法的入参中。



  **实验代码:**

```java
//@PathVariable 注解可以将请求URL路径中的请求参数，传递到处理请求方法的入参中
@RequestMapping(value="/testPathVariable/{id}")
public String testPathVariable(@PathVariable("id") Integer id){
System.out.println("testPathVariable...id="+id);
return "success";
}
```



<br>

# REST

---

## REST是什么？

REST：即 Representational State Transfer。**（资源）表现层状态转化。是目前最流行**的一种互联网软件架构。它结构清晰、符合标准、易于理解、扩展方便，所以正得到越来越多网站的采用
- **资源（Resources）**：网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的存在。可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的 URI 。获取这个资源，访问它的URI就可以，因此 URI 即为每一个资源的独一无二的识别符。
- **表现层（Representation）**：把资源具体呈现出来的形式，叫做它的表现层（Representation）。比如，文本可以用 txt 格式表现，也可以用 HTML 格式、XML 格式、JSON 格式表现，甚至可以采用二进制格式
- **状态转化（State Transfer）**：每发出一个请求，就代表了客户端和服务器的一次交互过程。HTTP协议，是一个无状态协议，即所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生“状态转化”（State Transfer）而这种转化是建立在表现层之上的，所以就是 “表现层状态转化”
- 具体说，就是 HTTP 协议里面，四个表示操作方式的动词：**GET、POST、PUT、DELETE**。它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源

## URL风格

order/1  HTTP **GET** ：得到 id = 1 的 order   

order/1  HTTP **DELETE：**删除 id = 1的 order   

order    HTTP **PUT**：更新order   

order     HTTP **POST**：新增 order 

## HiddenHttpMethodFilter

浏览器 form 表单只支持 GET 与 POST 请求，而DELETE、PUT 等 method 并不支持，Spring3.0 添加了一个过滤器，可以将这些请求转换为标准的 http 方法，使得支持 GET、POST、PUT 与 DELETE 请求

配置:

```xml
<!-- 支持REST风格的过滤器：可以将POST请求转换为PUT或DELETE请求 -->
<filter>
<filter-name>HiddenHttpMethodFilter</filter-name>
<filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
<filter-name>HiddenHttpMethodFilter</filter-name>
<url-pattern>/*</url-pattern>
</filter-mapping>
```

<br>

# 处理请求数据

---



## @RequestParam注解

- 在处理方法入参处使用 @RequestParam 可以把请求参数传递给请求方法

- value：参数名

- required：是否必须。默认为 true, 表示请求参数中必须包含对应的参数，若不存在，将抛出异常

- defaultValue: 默认值，当没有传递参数时使用该值

### DEMO

**控制器**:

```java
/**
 * @RequestParam 注解用于映射请求参数
 *         value 用于映射请求参数名称
 *         required 用于设置请求参数是否必须的
 *         defaultValue 设置默认值，当没有传递参数时使用该值
 */
@RequestMapping(value="/testRequestParam")
public String testRequestParam(
@RequestParam(value="username") String username,
@RequestParam(value="age",required=false,defaultValue="0") int age){
System.out.println("testRequestParam - username="+username +",age="+age);
return "success";
}
```

**页面链接**:

```jsp
<!--测试 请求参数 @RequestParam 注解使用 -->
<a href="springmvc/testRequestParam?username=atguigu&age=10">testRequestParam</a>
```

## @CookieValue 注解

- 使用 @CookieValue 绑定请求中的 Cookie 值

- **@CookieValue** 可让处理方法入参绑定某个 Cookie 值

### **DEMO**

```java
//了解:@CookieValue: 映射一个 Cookie 值. 属性同 @RequestParam
@RequestMapping("/testCookieValue")
public String testCookieValue(@CookieValue("JSESSIONID") String sessionId) {
System.out.println("testCookieValue: sessionId: " + sessionId);
return "success";
}
```

## 使用POJO作为参数

- 使用 POJO 对象绑定请求参数值

- Spring MVC **会按请求参数名和 POJO 属性名进行自动匹配，自动为该对象填充属性值**。**支持级联属性**。如：dept.deptId、dept.address.tel 等

### DEMO

{% tabs , 1 %}

<!--tab 控制器-->
{% code lang:java %}

/**
 * Spring MVC 会按请求参数名和 POJO 属性名进行自动匹配， 自动为该对象填充属性值。
 * 支持级联属性
 *                 如：dept.deptId、dept.address.tel 等
 */
@RequestMapping("/testPOJO")
public String testPojo(User user) {
System.out.println("testPojo: " + user);
return "success";
}

{% endcode %}
<!--endtab-->

<!--tab 表单-->

{% code lang:jsp%}
<!-- 测试 POJO 对象传参，支持级联属性 -->

<form action=" testPOJO" method="POST">
username: <input type="text" name="username"/><br>
password: <input type="password" name="password"/><br>
email: <input type="text" name="email"/><br>
age: <input type="text" name="age"/><br>
city: <input type="text" name="address.city"/><br>
province: <input type="text" name="address.province"/>
<input type="submit" value="Submit"/>
</form>
{% endcode %}

<!--endtab-->

<!--tab 实体类-->

{% code lang:java%}
public class Address {

private String province;
private String city;

//get/set

}
public class User {
private Integer id ;
private String username;
private String password;

private String email;
private int age;

private Address address;

//get/set 
}
{% endcode %}}

<!--endtab-->

<!--tab 中文乱码问题-->

**如果中文有乱码，需要配置字符编码过滤器，且配置其他过滤器之前，如（HiddenHttpMethodFilter），否则不起作用。（思考method=”get”请求的乱码问题怎么解决的）**

{% code lang:xml%}

​	**<!-- 配置字符集 -->**

​	<filter>

​		<filter-name>encodingFilter</filter-name>

​		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

​		<init-param>

​			<param-name>encoding</param-name>

​			<param-value>UTF-8</param-value>

​		</init-param>

​		<init-param>

​			<param-name>forceEncoding</param-name>

​			<param-value>true</param-value>

​		</init-param>

​	</filter>

​	<filter-mapping>

​		<filter-name>encodingFilter</filter-name>

​		<url-pattern>/*</url-pattern>

​	</filter-mapping>

{% endcode %}



<!--endtab-->

{% endtabs %}

<br>

# 处理相应数据

---

## 输出模型种类

**1）** **ModelAndView**: 处理方法返回值类型为 ModelAndView 时, 方法体即可通过该对象添加模型数据 

**2）** **Map 及 Model**: 入参为 org.springframework.ui.Model、org.springframework.ui.ModelMap 或 java.uti.Map 时，处理方法返回时，Map 中的数据会自动添加到模型中。

## 处理模型数据之 ModelAndView

### ModelAndView介绍

- 控制器处理方法的返回值如果为 ModelAndView, 则**其既包含视图信息，也包含模型数据信息**。

- 添加模型数据:

  MoelAndView addObject(String attributeName, Object attributeValue)

  ModelAndView addAllObject(Map<String, ?> modelMap)

- 设置视图:

  void setView(View view)

  void setViewName(String viewName)

### DEMO

控制器:

```JAVA
/**
 * 目标方法的返回类型可以是ModelAndView类型
 *                 其中包含视图信息和模型数据信息
 */
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
System.out.println("testModelAndView");
String viewName = "success";
ModelAndView mv = new ModelAndView(viewName );
mv.addObject("time",new Date().toString()); //实质上存放到request域中 
return mv;
}
```

页面:

```jsp
<!--测试 ModelAndView 作为处理返回结果 -->
<a href="springmvc/testModelAndView">time: ${requestScope.time }</a>
```

## 处理模型数据之 Map(推荐,便于框架移植)

### Map介绍

Spring MVC 在内部使用了一个 org.springframework.ui.Model 接口存储模型数据

具体使用步骤:

1. **Spring MVC 在调用方法前会创建一个隐含的模型对象作为模型数据的存储容器**。
2. **如果方法的入参为 Map 或 Model 类型，Spring MVC 会将隐含模型的引用传递给这些入参。**
3. **在方法体内，开发者可以通过这个入参对象访问到模型中的所有数据，也可以向模型中添加新的属性数据**

### DEMO

控制器:

```java
//目标方法的返回类型也可以是一个Map类型参数（也可以是Model,或ModelMap类型）
@RequestMapping("/testMap")
public String testMap(Map<String,Object> map){ //【重点】
System.out.println(map.getClass().getName());
//org.springframework.validation.support.BindingAwareModelMap
map.put("names", Arrays.asList("Tom","Jerry","Kite"));
return "success";
}
```

页面:

```jsp

names: ${requestScope.names }
```

> **注意问题：Map集合的泛型，key为String,Value为Object,而不是String**  

测试参数问题:

```java
//目标方法的返回类型也可以是一个Map类型参数（也可以是Model,或ModelMap类型）
@RequestMapping("/testMap2")
public String testMap2(Map<String,Object> map,Model model,ModelMap modelMap){
System.out.println(map.getClass().getName());
map.put("names", Arrays.asList("Tom","Jerry","Kite"));
model.addAttribute("model", "org.springframework.ui.Model");
modelMap.put("modelMap", "org.springframework.ui.ModelMap");
 
System.out.println(map == model);
System.out.println(map == modelMap);
System.out.println(model == modelMap);
 
System.out.println(map.getClass().getName());
System.out.println(model.getClass().getName());
System.out.println(modelMap.getClass().getName());
 
/*
true
true
true
org.springframework.validation.support.BindingAwareModelMap
org.springframework.validation.support.BindingAwareModelMap
org.springframework.validation.support.BindingAwareModelMap
    */ 
return "success";
}
```

<br>

# 视图解析

---

## 试图和视图解析器

1. 请求处理方法执行完成后，最终返回一个 ModelAndView 对象。对于那些返回 String，View 或 ModeMap 等类型的处理方法，**Spring MVC 也会在内部将它们装配成一个 ModelAndView 对象**，它包含了逻辑名和模型对象的视图
2. Spring MVC 借助**视图解析器**（**ViewResolver**）得到最终的视图对象（View），最终的视图可以是 JSP ，也可能是 Excel、JFreeChart等各种表现形式的视图
3. 对于最终究竟采取何种视图对象对模型数据进行渲染，处理器并不关心，处理器工作重点聚焦在生产模型数据的工作上，从而实现 MVC 的充分解耦

## 视图解析器

1. SpringMVC 为逻辑视图名的解析提供了不同的策略，可以在 Spring WEB 上下文中**配置一种或多种解析策略**，**并指定他们之间的先后顺序**。每一种映射策略对应一个具体的视图解析器实现类。
2. 视图解析器的作用比较单一：将逻辑视图解析为一个具体的视图对象
3. 所有的视图解析器都必须实现 ViewResolver 接口

## 常用的视图解析器实现类

![](https://i.loli.net/2019/08/18/yDzNKSXJ9lB2TC6.png)

1. 程序员可以选择一种视图解析器或混用多种视图解析器

2. 每个视图解析器都实现了 Ordered 接口并开放出一个 order 属性，**可以通过 order 属性指定解析器的优先顺序**，**order  越小优先级越高**。

3. SpringMVC 会按视图解析器顺序的优先顺序对逻辑视图名进行解析，直到解析成功并返回视图对象，否则将抛出 ServletException 异常

4.  InternalResourceViewResolver

   JSP 是最常见的视图技术，可以使用 InternalResourceViewResolve作为视图解析器

   ![](https://i.loli.net/2019/08/18/Fgrp7Le82GDoTuy.png)



## Spring的表单标签

1. 通过 SpringMVC 的**表单标签**可以实现将模型数据中的属性和 HTML 表单元素相绑定，以实现表单数据**更便捷编辑和表单值的回显**
2.  **form 标签:** 
   - 一般情况下，**通过 GET 请求获取表单页面**，而**通过 POST 请求提交表单页面**，**因此获取表单页面和提交表单页面的 URL 是相同的**。
   - **只要满足该最佳条件的契约**，**form:form 标签就无需通过 action 属性指定表单提交的 URL**
   - 可以通过 **modelAttribute** 属性指定绑定的模型属性，若没有指定该属性，则默认从 request 域对象中读取 **command** 的表单 bean，如果该属性值也不存在，则会发生错误。
3. SpringMVC 提供了多个表单组件标签，如 <form:input/>、<form:select/> 等，用以绑定表单字段的属性值，它们的共有属性如下：
   - **path**：**表单字段，对应 html 元素的 name 属性，支持级联属性**
   - htmlEscape：是否对表单值的 HTML 特殊字符进行转换，默认值为 true
   - cssClass：表单组件对应的 CSS 样式类名
   - cssErrorClass：表单组件的数据存在错误时，采取的 CSS 样式
4. **form:input、form:password、form:hidden、form:textarea**：对应 HTML 表单的 text、password、hidden、textarea 标签
5. **form:radiobutton**：单选框组件标签，当表单 bean 对应的属性值和 value 值相等时，单选框被选中
6. **form:radiobuttons**：单选框组标签，用于构造多个单选框
   - **items**：可以是一个 List、String[] 或 Map
   - **itemValue**：指定 radio 的 value 值。可以是集合中 bean 的一个属性值
   - **itemLabel**：指定 radio 的 label 值
   - **delimiter**：多个单选框可以通过 delimiter 指定分隔符
7. **form:checkbox**：复选框组件。用于构造单个复选框
8. **form:checkboxs**：用于构造多个复选框。使用方式同 form:radiobuttons 标签
9. **form:select**：用于构造下拉框组件。使用方式同 form:radiobuttons 标签
10.  **form:option**：下拉框选项组件标签。使用方式同 form:radiobuttons 标签
11. **form:errors**：显示表单组件或数据校验所对应的错误*
    - *<form:errors path= “*  ” /> ：显示表单所有的错误
    - <form:errors path= “user*” /> ：显示所有以 user 为前缀的属性对应的错误
    -  <form:errors **path= “username”** /> ：显示特定表单对象属性的错误