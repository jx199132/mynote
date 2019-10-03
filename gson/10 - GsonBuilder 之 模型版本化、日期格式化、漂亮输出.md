# 模型版本化

Gson可以通过@Since注解以及@Until注解来为你的Java对象设置版本控制，如此，则你的模型类里面被以上两个注解标记了的成员变量，将只有符合特定版本范围内时才会被序列化和反序列化。

这两个注解只有在通过GsonBuilder创建的Gson实例上才有效，我们需要通过GsonBuilder.setVersion(double)来激活。

## @Since
该注解指示出某一成员或类型在这一特定的版本号之后才存在。例如有下面的模板类：

```
public class User {
   private String firstName;
   private String lastName;
   @Since(1.0) private String emailAddress;
   @Since(1.0) private String password;
   @Since(1.1) private Address address;
 }
```

如果你使用new Gson()创建Gson实例，那么toJson()和fromJson()不会使用它们。

然而，如果你使用Gson gson = new GsonBuilder().setVersion(1.0).create()来创建Gson实例，那么toJson()和fromJson()方法将排除address域，因为它的版本号被设置为了1.1。

## @Until
@Until注解为某一成员或类型指定了一个版本号，代表该成员或类型在该版本号之前才存在。 
稍微改变User模型：

```
public class User {
   private String firstName;
   private String lastName;
   @Until(1.1) private String emailAddress;
   @Until(1.1) private String password;
 }
```
现在，我们使用Gson gson = new GsonBuilder().setVersion(1.2).create()来创建Gson实例，toJson()和fromJson()方将排除emailAddress和password域，这是因为传入的版本号为1.2，超过了我们给定的1.1。


# 格式化日期和时间

GsonBuilder 提供了三个重载
```
setDateFormat(String pattern):pattern遵循SimpleDateFormat类的惯例。
setDateFormat(int style)：style必须是DateFormat的一个常量。
setDateFormat(int dateStyle, int timeStyle)：类似于上面的，只不过将日期和时间分开了。
```

# 漂亮输出
序列化得到的JSON是无空格的，所有字符都密密麻麻挤在了一起，这虽然节约空间，但对于人的理解却不友好。只需相应的设置GsonBuilder即可：


```
Gson gson = new GsonBuilder().setPrettyPrinting().create();
String jsonOutput = gson.toJson(someObject);
```
