---
layout:     post
title:      web基础知识扫盲_Servlet_XML_Jsp
subtitle:   包含Servlet、XML、Jsp及其内置对象等内容。
date:       2020-02-17
author:     LJJ
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - web

---

# web基础知识

引出：**什么是Java web？**

- Java web是用Java来解决相关web互联网领域的技术、需要在特定的web服务器上运行，分为web服务器和web客户端两部分。能够跨平台，在多个不同平台下部署和运行。

**Java web需要学会哪些技术？**

- 基于页面的前端技术：包括HTML、CSS、JavaScript、jQuery等。
- 动态语言技术：Java、JSP等。
- 数据库技术：oracle、mysql、sqlserver等。
- 其他工具组件：服务器、SSM、SSH框架。

**Java web的应用场景？**

- 2004年，淘宝由PHP语言转换为Java语言。
- 铁路12306：高并发、数据量大。

### JSP

#### 简介

- JSP，即Java server pages，Java服务器页面，其根本是一个简化的Servlet设计。
- JSP是在传统的网页HTML文件中插入Java程序段(Scriptlet)和JSP标记(tag)，从而形成JSP文件，后缀名为(*.jsp)。

#### JSP与Servlet的区别

- JSP是Servlet的一种简化，可以方便地编写动态网页。
- Servlet完全是由Java程序代码构成流程控制和事务处理。
- Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML里分离开来。
- JSP侧重于视图，Servlet主要用于逻辑控制。

#### Eclipse中动态web项目结构

- 配置环境
  - jdk1.8
  - tomcat9.0

- 其项目结构图如下：

  <img src="https://i.loli.net/2020/02/18/Y5e9U4RkxHvgFiM.png" alt="3.PNG" style="zoom:80%;" />

- 细则：
  - Java Resources:包含项目的Java源代码。
  - WebContent:包含所有web资源，html、jsp、图像文件等
  - WEB-INF:包含web.xml文件以及classes和lib目录。

- JSP程序执行过程：

  ![2.PNG](https://i.loli.net/2020/02/18/wjtuSqsIrWa2nPz.png)

- 其生命周期：

  ![1.PNG](https://i.loli.net/2020/02/18/jQ9MTqeciB8EIbA.png)

#### JSP基本语法

- 语法格式：

  ```jsp
  <%out.println("heloworld");%>
  ```

- 程序脚本：语句块可以自由地将Java脚本与页面代码组合使用；声明的变量是局部变量。

- 注释：

  ```jsp
  <%--
  
  	<%out.println("heloworld");>
  
  --%>
  ```

- 内容输出表达式：

  ```jsp
  <%int i=5;%>
  <%=i%>
  ```

- 导包格式：

  ```jsp
  <%@page import="java.util.*" %>
  
```
  

#### 内置对象

- 什么叫内置对象：
  - JSP中不需要预先声明就可以在代码和表达式中随意使用。
- 内置对象有哪些，分别是用来干什么的：
  - request对象：代表来自客户端的请求，例如FORM表单中填写的信息，是最常用的对象。客户端的请求信息被封装在request对象中，通过它才能了解到客户的需求，然后做出响应。是HttpServeletRequest类的实例。
    - 方法getParameter()；获取表单提交信息。
    - 方法getProtocol()；获取用户使用的协议。
    - 方法getServeletPath()；获取用户提交信息的页面。
  - response对象：代表服务端对客户端的相应，通过response对象来组织发送到客户端的数据，需要向客户端发送文字时直接使用。
    - 方法sendRedirect()；重新定向客户端的请求。
    - 方法setContentType()；设置相应的MIME类型。
  - session对象：客户端与服务端的一次会话，从客户端连接到服务器的一个WebApplication开始，知道客户端与服务器断开链接为止。
    - 方法getId()；返回session创建时jsp引擎为他设置的唯一ID号。
    - 方法invalidate()；取消session，使session不可用。
  - out对象：是JspWritter类的实例，向客户端输出内容的常用对象
    - 方法clear()；清空缓存区的内容。
    - 方法close()；关闭输出流。
  - page对象：是java.lang.Object类的实例。代表了正在运行的由JSP文件产生的类对i象。
    - 方法getClass()；返回此Object类。
    - 方法notify()；唤醒一个等待的线程。
    - 方法enterMonitor()；对Object进行加锁。
  - application对象：实现了用户间的数据共享，可存放全局变量，开始于服务器的启动直到服务器的关闭。
    - 方法getAttribute();返回给定名的属性值。
  - pageContext对象：提供了对JSP页面所有的对象，可以访问到本页所在的session，也可以取本页面所在的application的某一属性值。
  - config对象：在Servelet初始化时，JSP引擎向他传递信息，包括属性名和属性值，以及服务器的有关信息。
  - exception对象：异常抛出，实际上是java.lang.Throwable的对象。
    - getMassage();返回描述异常的消息。
    - toString()；返回异常的描述信息。

### XML

- XML 是可扩展标记语言（extensible markup language）。
- XML 被设计用来传输和存储数据，其焦点是数据的内容。
- 通过 XML，数据能够存储在独立的 XML 文件中。使用户可以专注于使用 HTML/CSS 进行显示和布局，并确保修改底层数据不再需要对 HTML 进行任何的改变。
- 补充一个[菜鸟教程链接](https://www.runoob.com/xml/xml-dom.html)，这里不做过多描述和copy。

### Servlet

- 是Java Servelet的简称，也叫小服务程序或服务连接器。用Java编写的服务端程序，主要功能在于交互地游览和修改数据，生成动态Web内容。
- 生命周期：init()--services()--destroy()
- 请求：HttpServeletRequest;响应：HttpServeletResponse。
- 转发与重定向的区别：
  - 实现的对象不同，分别是HttpServeletRequest/HttpServeletResponse。
  - 转发时游览器的url地址栏不会发生改变，重定向时游览器的url地址会发生改变。
  - 转发时只请求服务器一次，重定向则两次。
- 同时附上[菜鸟教程Servlet](https://www.runoob.com/servlet/servlet-tutorial.html)

























