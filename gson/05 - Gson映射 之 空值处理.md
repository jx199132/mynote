
## Gson对空值默认过滤掉，不被输出
UserSimple模型
```
public class UserSimple {  
    String name;
    String email;
    boolean isDeveloper;
    int age;
}
```
测试代码（这里实例化UserSImple的时候email字段为空）

```
Gson gson = new Gson();  
UserSimple user = new UserSimple("Norman", null, 26, true);  
String usersJson = gson.toJson(user); 
```

输出结果（可以发现空字段不被输出）

```
{
  "age": 26,
  "isDeveloper": true,
  "name": "Norman"
}
```

## 强制输出空值
修改测试代码

```
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.serializeNulls();  
Gson gson = gsonBuilder.create();

UserSimple user = new UserSimple("Norman", null, 26, true);  
String usersJson = gson.toJson(user);  
System.out.println(usersJson);
```

测试结果（可以看到空值也被输出了）


```
{
  "age": 26,
  "email": null,
  "isDeveloper": true,
  "name": "Norman"
}
```
