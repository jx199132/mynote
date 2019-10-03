
# 准备工作
- 新建maven项目引入gson依赖（这里版本是2.8.2）
```
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.8.2</version>
    </dependency>
```
- 新建JavaBean
```
public class UserSimple {  
	
    private String name;
    private String email;
    private int age;
    private boolean isDeveloper;
    
	public UserSimple() {
		super();
	}
	public UserSimple(String name, String email, int age, boolean isDeveloper) {
		super();
		this.name = name;
		this.email = email;
		this.age = age;
		this.isDeveloper = isDeveloper;
	}
	
	@Override
	public String toString() {
		return "UserSimple [name=" + name + ", email=" + email + ", age=" + age + ", isDeveloper=" + isDeveloper + "]";
	}
	//get === set  省略
}
```


## 序列化（对象转成json字符串）
- 测试方法

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

- 测试结果

```
{"name":"Norman","email":"norman@futurestud.io","age":26,"isDeveloper":true}
```


## 反序列化（json字符串转成对象）
- 测试方法

```
    @Test
	public void test02() {
		String userJson = "{'age':26,'email':'norman@futurestud.io','isDeveloper':true,'name':'Norman'}"; 
		Gson gson = new Gson();  
		UserSimple userObject = gson.fromJson(userJson, UserSimple.class);
		System.out.println(userObject);
	}
```

- 测试结果

```
UserSimple [name=Norman, email=norman@futurestud.io, age=26, isDeveloper=true]

```
