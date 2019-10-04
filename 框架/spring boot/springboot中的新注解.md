# @RestController
通俗的说就是@RestController = @Controller + @ResponseBody。

在Spring MVC4之后，我们可以使用@RestController 注解来开发基于Spring MVC4的REST风格的JSON服务。

@Controller和@RestController的区别：

如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp页面，配置的视图解析器InternalResourceViewResolver不起作用，返回的内容就是Return 里的内容。

例如：本来应该到success.jsp页面的，则其显示success.

# http组合注解
Spring4.3中引进了

- @GetMapping
- @PostMapping
- @PutMapping
- @DeleteMapping
- @PatchMapping

来帮助简化常用的HTTP方法的映射，并更好地表达被注解方法的语义。

```
@GetMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.GET)

@PostMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.POST)

@PutMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.PUT)

@DeleteMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.DELETE)
```
