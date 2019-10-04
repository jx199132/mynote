
# freemarket支持

1 pom文件中引入freemarker

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

2 controller类

```
package com.jx.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class HelloController {
	
	@RequestMapping("showname")
	public ModelAndView helloFreemarket(String name) {
		ModelAndView mode = new ModelAndView("showname");
		mode.addObject("name", name);
		return mode;
	}
}
```

3 freemarker模板在src/main/templates下新建  showname.ftl


```
<title>freemarker</title>
<h1>hello ${name}</h1>
<div>
	测试 freemarker
</div>
```



4 测试
http://127.0.0.1:8080/showname?name=%E5%BC%A0%E4%B8%89

# thymeleaf
1 pom文件中引入thymeleaf

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
2 controller类

```
@RequestMapping("index")
public String index(Model model , String name) {
	System.out.println("index in");
	model.addAttribute("name", name);
	return "index";
}
```

3 thymeleaf模板在src/main/templates下新建  index.html


```
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>model</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="${name}"></p>
<p th:text="'Hello！, ' + ${name} + '!'" ></p>
</body>
</html>
```



4 测试
http://127.0.0.1:8080/index?name=%E5%BC%A0%E4%B8%89


以上两种模板可以共存：
https://github.com/jx199132/spring-boot-web-template.git