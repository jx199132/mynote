# 目标
通过spring-data-jpa完成 对用户的 一些操作

## 定义查询方法

**Junit**

![这里写图片描述](http://img.blog.csdn.net/20170427172224658?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**service**

![这里写图片描述](http://img.blog.csdn.net/20170427172323363?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170427172330702?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**Repository**

![这里写图片描述](http://img.blog.csdn.net/20170427172430380?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**目录结构**

![这里写图片描述](http://img.blog.csdn.net/20170427172522250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 结论

框架在进行方法名解析时，会先把方法名多余的前缀截取掉，比如 find、findBy、read、readBy、get、getBy，然后对剩下部分进行解析。并且如果方法的最后一个参数是 Sort 或者 Pageable 类型，也会提取相关的信息，以便按规则进行排序或者分页查询。

在查询时，通常需要同时根据多个属性进行查询，且查询的条件也格式各样（大于某个值、在某个范围等等），Spring Data JPA 为此提供了一些表达条件查询的关键字，大致如下：

```
And --- 等价于 SQL 中的 and 关键字，比如 findByUsernameAndPassword(String user, Striang pwd)；
Or --- 等价于 SQL 中的 or 关键字，比如 findByUsernameOrAddress(String user, String addr)；
Between --- 等价于 SQL 中的 between 关键字，比如 findBySalaryBetween(int max, int min)；
LessThan --- 等价于 SQL 中的 "<"，比如 findBySalaryLessThan(int max)；
GreaterThan --- 等价于 SQL 中的">"，比如 findBySalaryGreaterThan(int min)；
IsNull --- 等价于 SQL 中的 "is null"，比如 findByUsernameIsNull()；
IsNotNull --- 等价于 SQL 中的 "is not null"，比如 findByUsernameIsNotNull()；
NotNull --- 与 IsNotNull 等价；
Like --- 等价于 SQL 中的 "like"，比如 findByUsernameLike(String user)；
NotLike --- 等价于 SQL 中的 "not like"，比如 findByUsernameNotLike(String user)；
OrderBy --- 等价于 SQL 中的 "order by"，比如 findByUsernameOrderBySalaryAsc(String user)；
Not --- 等价于 SQL 中的 "！ ="，比如 findByUsernameNot(String user)；
In --- 等价于 SQL 中的 "in"，比如 findByUsernameIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；
NotIn --- 等价于 SQL 中的 "not in"，比如 findByUsernameNotIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；
```

## 声明自定义查询

**在junit加上如下代码**

![这里写图片描述](http://img.blog.csdn.net/20170427173840228?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**在server和实现类分别加上代码**

![这里写图片描述](http://img.blog.csdn.net/20170427173911979?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170427173921444?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**在UserRepository加上如下代码**

![这里写图片描述](http://img.blog.csdn.net/20170427173928929?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通过query注解来进行声明式查询 可以自己定义一些查询方式.

## 混合自定义功能
有些时候repository所提供的功能 无法使用 spring data的方法命名约定来描述，甚至 无法使用@query注解来实现，那么就需要自定义了.


**junit**

![这里写图片描述](http://img.blog.csdn.net/20170427174627489?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


**server 和 实现类**

![这里写图片描述](http://img.blog.csdn.net/20170427174706896?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170427174712725?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

到这里  并不是需要给  UserRepository  也添加一个方法blendQuery
而是新建一个 接口 定义此方法

```
package com.jx.springinaction.repository;

import com.jx.springinaction.bean.User;

public interface UserRepositoryPlug {
	User blendQuery(String id);
}
```

UserRepository  接口 继承  UserRepositoryPlug 接口

![这里写图片描述](http://img.blog.csdn.net/20170427174959138?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

新建实现类

```
package com.jx.springinaction.repository.impl;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

import com.jx.springinaction.bean.User;
import com.jx.springinaction.repository.UserRepositoryPlug;

public class UserRepositoryImpl implements UserRepositoryPlug{
	
	@PersistenceContext
	private EntityManager entityManager;
	
	@Override
	public User blendQuery(String id) {
		return (User) entityManager.createQuery("FROM User WHERE id = ?1").setParameter(1, id).getResultList().get(0);
	}

}
```
注意这里继承的是 Plug接口 ，如果继承 UserRepotirory 接口 就会实现N多方法

当spring data jpa 查找 repository 的时候   比如查找到  UserRepository  继承  JpaRepository ，还会查找  此名字  加上 Impl后缀的类  ，   如果不想以Impl为后缀，可以修改repository-impl-postfix 来 配置

```
<jpa:repositories base-package="com.jx.springinaction.repository" repository-impl-postfix=""/>
```