
# Enum 序列化
先来定义一个枚举 Day
```
public enum Day {  
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY
}
```
创建一个 Java model 类

```
public class UserDayEnum {  
    private String _name;
    private String email;
    private boolean isDeveloper;
    private int age;

    private Day day = Day.FRIDAY;
}


```
使用 Gson 来序列化 UserDayEnum 对象
```
UserDayEnum userObject = new UserDayEnum("Norman", "norman@fs.io", true, 26, Day.SUNDAY);

Gson gson = new Gson();  
String userWithEnumJson = gson.toJson(userObject);


```
输出：

```
{
      "_name": "Norman",
      "age": 26,
      "day": "SUNDAY",
      "email": "norman@fs.io",
      "isDeveloper": true
}
```
根据结果我们看到，不用做任何配置处理，Gson 就帮我们正常输出了 JSON 格式的数据。

# Enum 反序列化
反序列化也非常简单，同样不做任何额外的配置：

```
String userJson = "{\"age\":26,\"email\":\"norman@futurestud.io\",\"isDeveloper\":true,\"day\":\"FRIDAY\"}";
    Gson gson = new Gson();
    UserDayEnum userObject = gson.fromJson(userJson, UserDayEnum.class);

```
输出
```
userDayEnum:UserDayEnum{_name='null', email='norman@futurestud.io', isDeveloper=true, age=26, day=FRIDAY}

```
# 自定义枚举(反)序列化

```
public enum Day2 {
    @SerializedName("0")
    MONDAY(),

    @SerializedName("1")
    TUESDAY,

    @SerializedName("2")
    WEDNESDAY,

    @SerializedName("3")
    THURSDAY,

    @SerializedName("4")
    FRIDAY,

    @SerializedName("5")
    SATURDAY,

    @SerializedName("6")
    SUNDAY
}

```
