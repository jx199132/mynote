# 说明 
JSON 规范的第2.4节不允许特殊的double值（NaN，Infinity，-Infinity）,但是，Javascript规范（见第4.3.20,4.3.22,4.3.23节）允许这些值作为有效的 Javascript 值。 此外，大多数 JavaScript 引擎将接受 JSON 中的这些特殊值，而没有问题。 因此，在实际应用中，即使不能作为 JSON 规范，但是接受这些值作为有效的 JSON 是有意义的。


```
public class UserFloat {  
    String name;
    Float weight;

    public UserFloat(String name, Float weight) {
        this.name = name;
        this.weight = weight;
    }
}
```

测试代码：

```
UserFloat user = new UserFloat("Norman", 12.6f);  
Gson gson = new Gson();

String usersJson = gson.toJson(user);
System.out.println(usersJson);
```
结果 （这样正常的数据12.6f 是没问题的）：

```
{"name":"Norman","weight":12.6}

```

测试代码：

```
UserFloat user = new UserFloat("Norman", Float.POSITIVE_INFINITY);  
Gson gson = new Gson();

String usersJson = gson.toJson(user); 

```
结果（抛出异常）：

```

java.lang.IllegalArgumentException: Infinity is not a valid double value as per JSON specification. To override this behavior, use GsonBuilder.serializeSpecialFloatingPointValues() method.
```

解决方式：

```
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.serializeSpecialFloatingPointValues();
Gson gson = gsonBuilder.create();
UserFloat userFloat = new UserFloat("Norman", Float.POSITIVE_INFINITY);
String usersJson = gson.toJson(userFloat);
System.out.println("userJson:" + usersJson);
```
输出结果：

```
userJson:{"name":"Norman","weight":Infinity}
```
