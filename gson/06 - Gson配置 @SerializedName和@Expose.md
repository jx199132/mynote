
## @Expose 的 transient
测试代码

```
UserSimple userObject = new UserSimple(  
			    "Norman", 
			    "norman@futurestud.io", 
			    26, 
			    true
			);
		
		Gson gson = new Gson();
		String userJson = gson.toJson(userObject);
		System.out.println(userJson);
```
UserSimple类
```
    private String name;
	
    private String email;
    
    private int age;
    
    private boolean isDeveloper;
```
正常情况下jsoon四个字段都存在

```
{"name":"Norman","email":"norman@futurestud.io","age":26,"isDeveloper":true}
```

在UserSimple类修改一下 email 字段加上 transient修饰

```
    private String name;
	
    private transient String email;
    
    private int age;
    
    private boolean isDeveloper;
```
输出结果 ： 可以看到 email字段就被忽略了

```
{"name":"Norman","age":26,"isDeveloper":true}
```

## @Expose （注意这里的测试代码Gson是通过builder生成）
测试代码

```
    @Test
	public void test05() {
		UserSimple userObject = new UserSimple(  
				"Name", 
				"norman@futurestud.io", 
				26, 
				true
				);
		GsonBuilder builder = new GsonBuilder();  
		builder.excludeFieldsWithoutExposeAnnotation();  
		Gson gson = builder.create();  
		String userJson = gson.toJson(userObject);
		System.out.println(userJson);
	}
```
UserSimple类
```

    private String name;
	
    private String email;
    
    private int age;
    
    private boolean isDeveloper;

```
这种情况下输出是不会有任何数据的，因为把所有的字段都排除了。

如果需要不排除某些字段，那么需要在该字段加上@Expose注解如下代码


```
    @Expose
    private String name;
	
	@Expose
    private String email;
    
	@Expose
    private int age;
    
    private boolean isDeveloper;
```

输出结果

```
{"name":"Name","email":"norman@futurestud.io","age":26}
```

说明：
Expose注解有两个参数，serialize和deserialize 值为 true / false

## @SerializedName注解

该注解用于对字段名进行改变或者补充，有两个参数value和alternate，第一个参数必填，第二个选填。

value改变了==序列化和反序列化==的默认情况！因此，如果Gson根据你的Java模型类创建了一个JSON，它将会使用value作为该属性的名。

alternate==仅仅是作为反序列化==中的代选项。Gson将会JSON中的所有名称并且尝试映射到被注解了的属性中的某一个。在上面的模型类中，Gson将会检查到来的JSON中是否含有fullName或者username。无论是哪一个，都会映射到name属性


测试代码
```
    @Test
	public void test01() {
		UserSimple userObject = new UserSimple(  
			    "Norman", 
			    "norman@futurestud.io", 
			    26, 
			    true
			);
		
		Gson gson = new Gson();
		String userJson = gson.toJson(userObject);
		System.out.println(userJson);
	}
```
UserSimple类

```
    private String name;
	
    private String email;
    
    private int age;
    
    private boolean isDeveloper;
```
正常情况下输出内容：

```
{"name":"Norman","email":"norman@futurestud.io","age":26,"isDeveloper":true}
```

需求：想改变输出字段，比如name字段在json输出的时候叫做fullname

那么如下：可以在name字段上加上注解输入值即可
```
    @SerializedName("fullname")
    private String name;
	
    private String email;
    
    private int age;
    
    private boolean isDeveloper;
    
	public UserSimple() {
		super();
	}
```

```
{"fullname":"Norman","email":"norman@futurestud.io","age":26,"isDeveloper":true}
```







