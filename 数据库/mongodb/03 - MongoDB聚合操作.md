## **说明**

MongoDB中聚合(aggregate)主要用于处理数据(诸如统计平均值,求和等)，并返回计算后的数据结果

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。



## **Aggregate**

### 示例

```java
@Test
public void testAggregate1(){
    coll.aggregate(Arrays.asList(
                Aggregates.match(Filters.eq("sex", 1)),//查询所有sex=1的数据
                Aggregates.group("$age", Accumulators.sum("total", 1)),//根据age分组 , 设置求和的字段名为total
                Aggregates.sort(new Document().append("total", 1))//根据total进行排序 1 升 , -1降
            )).forEach(new Block<Document>() {//通过foreach打印
                @Override
                public void apply(Document document) {
                    System.out.println(document.toJson());
                }
            });
```



### 聚合表达式

```
$sum   计算总和。   
$avg   计算平均值   
$min   获取集合中所有文档对应值得最小值。
$max   获取集合中所有文档对应值得最大值。
$push  在结果文档中插入值到一个数组中。    
$addToSet  在结果文档中插入值到一个数组中，但不创建副本。 
$first 根据资源文档的排序获取第一个文档数据。 
$last  根据资源文档的排序获取最后一个文档数据
```



### 更多

官网有很多详细

http://mongodb.github.io/mongo-java-driver/3.4/javadoc/?com/mongodb/client/model/Accumulators.html 



## **聚合管道**

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。

**聚合框架中常用的几个操作：**

```
$project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
$match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
$limit：用来限制MongoDB聚合管道返回的文档数。
$skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
$unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
$group：将集合中的文档分组，可用于统计结果。
$sort：将输入文档排序后输出。
$geoNear：输出接近某一地理位置的有序文档。
```



```java
@Test
public void testAggregate2(){
    coll.aggregate(Arrays.asList(
                Aggregates.match(Filters.eq("sex", 1)),//查询所有sex=1的数据
                Aggregates.project(new Document("age",1))//0不显示age ,1 只显示age  0等价于false  1 等价于true
            )).forEach(new Block<Document>() {//通过foreach打印
                @Override
                public void apply(Document document) {
                    System.out.println(document.toJson());
                }
            });
}
```

