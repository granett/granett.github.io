---
layout: post
title: "Spring boot入门之搭建项目"
date: 2019-09-09
tag: 框架
---   

### **什么是Spring boot？**
从最根本上来讲，Spring Boot就是一些框架和配置的集合，就比如**Spring相当于把各种类型的bean组织在一起一样，Spring boot就是把各种框架和各种配置整合在了一起，比如Spring、Spring MVC还有redis、mongo等，**它能够被任意项目的构建系统所使用。
### Spring boot的优点
>**1.使用 Spring 项目引导页面可以在几秒构建一个项目**        
>**2.集成了各种关系数据库和非关系数据库**        
>**3.嵌入的Tomcat，无需部署WAR文件**        
>**4.简化和自动配置Spring等各种框架**        
>**5.很大程度上精简了复杂的XML配置**        

### **生成项目**
1.访问http://start.spring.io/，这里是Spring 项目引导页面
2.选择构建工具Maven Project、Spring Boot版本以及输入项目的结构等
![QQ图片20180125113718.png](/images/1.jpg)
3.点击上图下方的`Switch to the full version`，**可以选择需要添加的数据库、框架等等。**
![QQ图片20180125114031.png](/images/2.jpg)
4.点击`Generate Project`下载项目压缩包，**并把项目导入到开发工具。**

### **做一个简单的web后台接口**
1.首先我们需要在pom.xml里引入web依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
2.编写测试的接口类`HelloController `
```
package com.didispace.web;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }

}
```
3.启动主程序，打开浏览器访问`http://localhost:8080/hello`，可以看到页面输出`Hello World`
### **注意点**
**1.@RestController的意思就是该controller里面的方法都以json格式返回，不用再像SpringMVC里似的写@ResponseBody了。
2.我们可以看到pom.xml里的依赖也都和Spring里的名字不同了，这也是Spring boot精简配置的体现之一。**
**3.如果出现下图错误**
![QQ图片20180125115248.png](/images/3.jpg)
Spring Boot默认使用嵌入式Tomcat，默认没有页面来处理404等常见错误。**因此，为了给用户最佳的使用体验，404等常见错误需要我们自定义页面来处理，错误页面需要放在Spring Boot web应用的static内容目录下，它的默认位置是：src/main/resources/static，如下图所示：**

![QQ图片20180125115431.png](/images/4.jpg)
图片可以去这里下载：http://sporcic.org/wp-content/uploads/2014/05/error-pages.zip

到这里Spring boot的第一个小项目已经完成了。







