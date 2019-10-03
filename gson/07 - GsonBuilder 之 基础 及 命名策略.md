
## GsonBuilder 基础

Gson默认提供了一些功能，转换方式，使用方法如下：
```
Gson gson = new Gson();  
UserSimple userObject = gson.fromJson(userJson, UserSimple.class);
```

还提供了一种builder的方式

```
GsonBuilder gsonBuilder = new GsonBuilder();  
Gson gson = gsonBuilder.create();
```

前面简单在  空值强制输出和@Expose注解 中简单使用到了



```
    private String name;
	
    private String email;
    
    private int age;
    
    private boolean isDeveloper;
```

### 需求上面四个字段都是小写，需要把字段首字母都改成大写

如果使用@Serialized注解 那么需要对每一个字段都进行配置，那么太麻烦，Builder中提供了 一种方式可以实现

测试：

```
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE_WITH_SPACES);
Gson gson = gsonBuilder.create();

UserSimple user = new UserSimple("Norman", "norman@futurestud.io", 26, true);  
String usersJson = gson.toJson(user);  
System.out.println(usersJson);
```
输出结果：

```
{"Name":"Norman","Email":"norman@futurestud.io","Age":26,"Is Developer":true}
```

builder中提供了多种命名方式，也提供了自定义的方式。

## 自定义命名
通过builder的setFieldNamingStrategy方法来对字段名称全部转为大写


测试代码
```
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setFieldNamingStrategy(new FieldNamingStrategy() {
	public String translateName(Field f) {
		return f.getName().toUpperCase();
	}
});
Gson gson = gsonBuilder.create();

UserSimple user = new UserSimple("Norman", "norman@futurestud.io", 26, true);  
String usersJson = gson.toJson(user);  
System.out.println(usersJson);
```
结果

```
{"NAME":"Norman","EMAIL":"norman@futurestud.io","AGE":26,"ISDEVELOPER":true}
```


以上方式在反序列化一样有效
