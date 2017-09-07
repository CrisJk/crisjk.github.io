---
layout: post
title: 使用Spring boot 创建web工程 
author: Kuang
tags: springboot
categories: web
---

利用Springboot新建一个web工程









方法有很多种，一种比较方便的方法就是直接访问[Spring INITIALIZR][1],填好信息，然后Alt + Enter键直接生成即可。然后使用IDE（对不起我比较low）import刚才生成的项目，我是使用maven方式导入的。导入完之后进到pom.xml里面保存一下，maven就会自动地下载一些需要的包(写在dependency里面)...

之前的都是废话，是个人应该都会。创建项目还有一个重要的步骤就是定义项目结构，我~~参考~~照抄了[Serving Web Content with Spring MVC][2]  ，然后采用MVC模式建立的工程.

**Controller层**

`/src/main/java/com/ecnu/argiculture/userprofile/UserprofileApplication.java`

```java
package com.ecnu.argiculture.userprofile.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * Created by kuangjun on 9/6/17.
 */
@Controller
public class TryController {
        @RequestMapping("/index")
        public String index(@RequestParam(value="name",required=false,defaultValue="ECNUer") String name, Model model){
                model.addAttribute("name",name);
                return  "I don't know what to return";
        }

}
```

controller层可以理解为连接前后端的一层，它利用addAttribute的方式将后端的一些数据传送给前端. 其中，注解@Controller定义了该类是作为Controller. @RequestMapping确保当你在浏览器访问/index的地址时，对应的是index()这个方法

> 这里并没有指定HTTP方法(GET PUT POST etc.)，这样默认就能使用所有的方法。当然，也可以使用@RequestMapping(method=GET)之类的来限定方法

@RequestParam将前段页面请求的String类型参数`name`与`index()`方法中的`name`参数绑定，这个`name`参数的值并不是必须填写的(`required = false`)，如果未填写的话,那么`name`的值就等于`defaultValue`即`ECNUer` 最后，通过`model.addAttribute()`将`name`参数加入到`Model`对象中，以使得其能被页面获得



然后有一种叫做[Thymeleaf](http://www.thymeleaf.org/doc/tutorials/2.1/thymeleafspring.html)的东西，可用于渲染XML/XHTML/HTML5,它可以与Spring MVC结合作为web应用的模板引擎. Thymeleaf的特点是它能够直接在浏览器中打开模板页面，而不需要重新启动web应用. 使用Thymeleaf的另一大好处就是可以进一步做到前后端分离，它通过属性进行渲染，不会引入任何浏览器不能识别的标签，因此不需要在应用服务器中渲染也可以看到页效果. 

**页面**

上面已经实现了一个Controller,接下来希望可以有一些"看得见，摸得着"的东西，于是可以简单写一个页面

`/src/main/resources/templates/index.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Home page</title>
    <meta http-equiv="Content-Type" content="text/html ; charset = UTF-8"/>
</head>
<body>
    <p th:text=" 'Hello', + ${name} + '!' "></p>
</body>
</html>
```

> 注意那个th: text

**Spring-boot-devtools **

通常来说，当我们修改代码后，想要查看效果，必须要重启应用，然后刷新页面，这个过程会耗费我们大量的时间(比如我们经常在这个时候看个视频玩个游戏啥的~~),于是Spring boot提供了一个叫做`Spring-boot-devtools`的模块，让我们没必要每次都重启一下应用(于是少了很多娱乐时间，万恶的资本主义)，因为它会自动检测资源文件resourses的变动触发重启，这个重启速度比较快. 吧？ 默认情况下有些文件的改变会导致重启，有些不会，这个也可以自己配置.反正有这么个东西可以实现热部署.

要使用Spring-boot-devtools,只要在pom.xml里面添加相关依赖即可

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
</dependency>
```



**Application类**

现在有了一个前段页面和controller层，启动应用就能看到页面了.于是得有个开关吧，Application类就相当于这个应用的开关，执行这个类就能启动程序. 当然Application类这个名字是我瞎编的，你叫它ABC类也可以..

`/src/main/java/com/ecnu/argiculture/userprofile/UserprofileApplication.java`

```java
package com.ecnu.argiculture.userprofile;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class UserprofileApplication {

	public static void main(String[] args) {
		SpringApplication.run(UserprofileApplication.class, args);
	}
}
```

`@SpringBootApplication`是一种便捷地添加以下注解的方法:

* `@Configuration` 告诉你这是一个配置类，你可以用某些xml配置达到同样的效果.
* `@EnableAutoConfiguration` 告诉Springboot开始通过各类配置，加载对应的beans
* `@EnableWebMvc`
* `@ComponentScan`命令Spring扫描其他的组件、配置和服务，使其可以找到controllers

好了，接下来运行这个Application类，然后在浏览器输入localhost:8080/index，你就可以看到如下页面

 ![][3]

如果你在浏览器中输入localhost:8080/index?name=常山赵子龙,就会得到以下界面

![][4]

[1]:https://start.spring.io/
[2]: https://spring.io/guides/gs/serving-web-content/
[3]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/SpringStart-2.png
[4]:https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/SpringStart-1.png
