## **目标**

本章目标，通过Java驱动的方式操作MongoDB，进行基本增删改查 操作。



## **准备**

下载驱动，这里采用maven管理jar

```xml
  <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongo-java-driver</artifactId>
            <version>3.4.2</version>
        </dependency>
```



连接MongoDB

```java
package com.jx.mongdo;

import com.mongodb.MongoClient;

public class MongoDBUtil{

    private static MongoClient mongoClient = new MongoClient( "192.168.7.127", 27018);

    public static MongoClient getClient(){
        return mongoClient;
    }

}
```



### 写入数据

新建一个类，通过junit测试下面代码

```java
private MongoCollection<Document> coll;

@Before
public void before(){
  MongoClient client = MongoDBUtil.getClient();
  MongoDatabase db = client.getDatabase("mydb01");
  coll = db.getCollection("user");
}


@Test
public void testInsert(){
  Document doc = new Document();
  doc.append("hello", "mongoDB");
  coll.insertOne(doc);
}
```



### 修改数据

```java
@Test
public void testUpdate(){
    Document doc = new Document();
    doc.put("hello", "mongoDB");
    Document ndoc = new Document();
    ndoc.put("mongDB", "hello");
    coll.replaceOne(doc, ndoc);
}
```



### 删除数据

```java
@Test
public void testDel(){
  Document doc = new Document();
  doc.put("mongDB", "hello");
  coll.deleteOne(doc);
}
```



### 查询

#### **初始化一些测试数据**

这里初始化了1000条测试数据，过程就略过，数据结构如下

<img src="https://raw.githubusercontent.com/jx199132/pic/master/pic/20190927161837.png" style="zoom:50%;" />



#### 查询全部

```java
@Test
public void testQueryAll(){
    FindIterable<Document> ite = coll.find();//获取迭代器
    MongoCursor<Document> cursor = ite.iterator();//获取游标
    while(cursor.hasNext()){
        Document doc = cursor.next();
        System.out.println(doc.toJson());
    }
    cursor.close();
}
```



#### **等于查询**

```java
@Test
public void testQueryEqual(){
    MongoCursor<Document> cursor = coll.find(Filters.eq("address", "蛟河市")).iterator();
    while(cursor.hasNext()){
        Document doc = cursor.next();
        System.out.println(doc.toJson());
    }
    cursor.close();
}
```



#### **查询小于**

```java
@Test
public void testQueryLt(){
    MongoCursor<Document> cursor = coll.find(Filters.lt("age", 11)).iterator();
    while(cursor.hasNext()){
        Document doc = cursor.next();
        System.out.println(doc.toJson());
    }
    cursor.close();
}
```



```
$lt  小于
$lte 小于或等于
$gt  大于
$gte 大于或等于
$ne  不等于
```



#### **And查询**

```java
@Test
public void testQueryAnd(){
    MongoCursor<Document> cursor = coll.find(Filters.and( Filters.eq("age", 11),Filters.eq("sex", 1) )).iterator();
    while(cursor.hasNext()){
        Document doc = cursor.next();
        System.out.println(doc.get("age") + " : " + doc.get("sex"));
    }
    cursor.close();
}
```

#### **like查询**

在MongoDB中like查询是通过正则的形式

```java
@Test
public void testQueryLike(){
    String where = "汽车";
    Pattern pattern = Pattern.compile("^.*" + where+ ".*$", Pattern.CASE_INSENSITIVE); 
    MongoCursor<Document> cursor = coll.find(Filters.regex("job", pattern)).iterator();
    while(cursor.hasNext()){
        Document doc = cursor.next();
        System.out.println(doc.toJson());
    }
    cursor.close();
}
```

#### **集合中查询**

mongoDB中支持数据中的查询，这里演示的是 查询 hoby中的元素 包含完全包含下面的数组数据

```java
@Test
public void testQueryArray(){
    MongoCursor<Document> cursor = coll.find(Filters.all("hoby", "天津工程职业技术学院","包头铁道职业技术学院")).iterator();
    while(cursor.hasNext()){
        Document doc = cursor.next();
        System.out.println(doc.toJson());
    }
    cursor.close();
}
```

#### **对象属性查询**

这里根据User下面的Dog 名称进行查询

```java
@Test
public void testQueryObject(){
    MongoCursor<Document> cursor = coll.find(Filters.eq("dog.name", "葛习厉")).iterator();
    while(cursor.hasNext()){
        Document doc = cursor.next();
        System.out.println(doc.toJson());
    }
    cursor.close();
}
```

#### **更多查询**

MongoDB支持很多查询方式，总结归纳如下： 

1 普通查询 key = value形式 

2 范围查询 lt,lt,gt lteltegte 小于，大于，小于等于，大于等于 

3 集合操作 in,in,all , nin上面集合中查询演示了all4布尔查询nin上面集合中查询演示了all4布尔查询ne notnotor andandexists 

5 子文档元素查询 上面例子中有提到根据User的Dog的名称查询 

6 数组查询 find(tags:’aaa’) 查询 所有 tags这个数组中存在 aaa这个元素的 

7 Javascript查询 

8 正则表达式查询 

9 其他查询 type(根据字段类型匹配)type(根据字段类型匹配)mod find({ age : { $mod : [3,0] } }) 查询年龄除以3余0的文档



下面是官网api地址

http://api.mongodb.com/java/3.0/?com/mongodb/client/model/Filters.html 



#### **其他查询需要注意的**



##### **查询部分字段**

只查询 三个字段

```java
@Test
public void testQueryalitter(){
    MongoCursor<Document> cursor = coll.find().projection(Projections.include("nickname","age","dog")).iterator();
    while(cursor.hasNext()){
        Document doc = cursor.next();
        System.out.println(doc.toJson());
    }
    cursor.close();
}
```

