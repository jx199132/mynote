# 比transient和@Expose更牛逼的排除策略

先创建一个测试模型。我们使用一个新的UserDate模型，它拥有一些属性：

```
public class UserDate {  
    private String _name;
    private String email;
    private boolean isDeveloper;
    private int age;
    private Date registerDate = new Date();
}
```

需要排除所有类型为Date以及boolean的属性，使用ExclusionStrategies很容易做到。你可以通过GsonBuilder实现：

```
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setExclusionStrategies(new ExclusionStrategy() {  
    @Override
    public boolean shouldSkipField(FieldAttributes f) {
        return false;
    }

    @Override
    public boolean shouldSkipClass(Class<?> incomingClass) {
        return incomingClass == Date.class || incomingClass == boolean.class;
    }
});
Gson gson = gsonBuilder.create();

UserDate user = new UserDate("Norman", "norman@futurestud.io", 26, true);  
String usersJson = gson.toJson(user); 
```
==ExclusionStrategies==类提供了两个重写方法。上面的例子我们使用了第二个方法。我们检查该类是否是Date或者boolean之一。如果该属性是其中之一的类型，那么该方法就会返回true，Gson将会忽略该属性。你可以在该方法中检查任意类。JSON结果将只包含字符串型和整型：

```
{
  "age": 26,
  "email": "norman@futurestud.io",
  "_name": "Norman"
}
```

另一个方法的排除功能基于字段的声明。例如，如果我们还想排除所有包含下划线_的字段，那么可以按如下代码：

```
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setExclusionStrategies(new ExclusionStrategy() {  
    @Override
    public boolean shouldSkipField(FieldAttributes f) {
        return f.getName().contains("_");
    }

    @Override
    public boolean shouldSkipClass(Class<?> incomingClass) {
        return incomingClass == Date.class || incomingClass == boolean.class;
    }
});
Gson gson = gsonBuilder.create();

UserDate user = new UserDate("Norman", "norman@futurestud.io", 26, true);  
String usersJson = gson.toJson(user);
```

JSON结果更短了：

```
{
  "age": 26,
  "email": "norman@futurestud.io"
}
```

## 排除策略仅作用于序列化或者反序列化
上面通过==setExclusionStrategies==排除策略==不仅仅用于序列化也用于反序列化==

通过下面两个方法可以精确用于序列化或者反序列化
```
addSerializationExclusionStrategy()
addDeserializationExclusionStrategy()
```

## 基于修饰词排除字段

```
public class UserModifier {  
    private String name;
    private transient String email;
    private static boolean isDeveloper;
    private final int age;
}
```
排除所有final和static类型，但包括field

```
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.excludeFieldsWithModifiers(Modifier.STATIC, Modifier.FINAL);  
Gson gson = gsonBuilder.create();

UserModifier user = new UserModifier("Norman", "norman@fs.io", 26, true);  
String usersJson = gson.toJson(user);
```

上面的代码将会创建如下JSON：

```
{
  "email": "norman@fs.io",
  "name": "Norman"
}
```
注意，Gson实例也会包含email域的，即使它被设置为transient。调用excludeFieldsWithModifiers()方法重写它的默认设置。我们仅仅传递了static和final，因此transient修饰词将不会被忽略。如果你想要包含所有域而不管修饰词，仅需要传递空列表参数给excludeFieldsWithModifiers()


