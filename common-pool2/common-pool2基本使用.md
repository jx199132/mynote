
## 自定义一个String工厂实现PooledObjectFactory

PooledObjectFactory管理具体对象的状态，比如创建，初始化，验证对象状态和销毁对象。

那么创建一个字符工厂类来实现PooledObjectFactory类，在创建，在各种状态的时候进行输出。

```
import java.util.UUID;
import org.apache.commons.pool2.PooledObject;
import org.apache.commons.pool2.PooledObjectFactory;
import org.apache.commons.pool2.impl.DefaultPooledObject;

public class StringFactory implements PooledObjectFactory<String>{
    public StringFactory(){
        System.out.println("init string factory..");
    }
    public void activateObject(PooledObject<String> pool) throws Exception {
    	
    }

    public void destroyObject(PooledObject<String> pool) throws Exception {
        String str = pool.getObject();
        if(str!=null){
            str=null;
            System.out.println(str+" destroy...");
        }
    }

    public PooledObject<String> makeObject() throws Exception {
        String i = UUID.randomUUID().toString();
        System.out.println("make "+i+" success...");
        return new DefaultPooledObject<String>(i);
    }

    public void passivateObject(PooledObject<String> pool) throws Exception {

    }

    public boolean validateObject(PooledObject<String> pool) {
        return false;
    }

}

```

## 测试方法1

通过 GenericObjectPool 类 获取 String 对象

```
    @Test
	public void t1() {
		GenericObjectPoolConfig conf = new GenericObjectPoolConfig();
        conf.setMaxTotal(10);
        GenericObjectPool<String> pool = new GenericObjectPool<String>(new StringFactory(), conf);
        for(int i=0;i<15;i++){
            System.out.println(i+":");
            try {
                String str = pool.borrowObject();
                System.out.println(str);
            } catch (Exception e) {
                e.printStackTrace();
            }
       }
	}
```

输出：

```
init string factory..
0:
make 2f5df2db-90ec-4b4c-8769-77a0f5620e6f success...
2f5df2db-90ec-4b4c-8769-77a0f5620e6f
1:
make 6d75f923-077d-47cf-a404-a389adbf14ae success...
6d75f923-077d-47cf-a404-a389adbf14ae
2:
make c5791ce2-c29c-4e6d-a0eb-be907ba4e6d4 success...
c5791ce2-c29c-4e6d-a0eb-be907ba4e6d4
3:
make 9a0b8311-2438-4930-8734-090875e226e5 success...
9a0b8311-2438-4930-8734-090875e226e5
4:
make 74da18ad-d148-4fed-873b-9a97e4c6a840 success...
74da18ad-d148-4fed-873b-9a97e4c6a840
5:
make 37b24023-6c3d-4bb8-9181-846e942f0d1a success...
37b24023-6c3d-4bb8-9181-846e942f0d1a
6:
make a5e281c1-3499-4782-ba27-5a731c60182b success...
a5e281c1-3499-4782-ba27-5a731c60182b
7:
make dc777ae5-6cd3-431a-9abe-fdc12e760d36 success...
dc777ae5-6cd3-431a-9abe-fdc12e760d36
8:
make 58d77d4e-7035-4e82-bc6b-c42d5bb4d4d5 success...
58d77d4e-7035-4e82-bc6b-c42d5bb4d4d5
9:
make 3b0003a1-7d28-4796-8c2a-27f05bdd7d17 success...
3b0003a1-7d28-4796-8c2a-27f05bdd7d17
10:
```

结论：

```
由于每次使用完毕都不进行归还，所以每一次都创建一个新的String对象
```


## 测试方法2

```
@Test
	public void t2() {
		GenericObjectPoolConfig conf = new GenericObjectPoolConfig();
        conf.setMaxTotal(10);
        GenericObjectPool<String> pool = new GenericObjectPool<String>(new StringFactory(), conf);
        for(int i=0;i<15;i++){
            System.out.println(i+":");
            try {
                String str = pool.borrowObject();
                System.out.println(str);
                boolean isReturn = new Random().nextBoolean();
                System.out.println(isReturn);
                if(isReturn) {
                	pool.returnObject(str);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
       }
	}
```

输出：

```
init string factory..
0:
make b1c3d369-c042-4a1d-ab92-b840207b88c8 success...
b1c3d369-c042-4a1d-ab92-b840207b88c8
false
1:
make 6da5b1ef-3b14-41f1-b9f6-847a6129566f success...
6da5b1ef-3b14-41f1-b9f6-847a6129566f
true
2:
6da5b1ef-3b14-41f1-b9f6-847a6129566f
true
3:
6da5b1ef-3b14-41f1-b9f6-847a6129566f
false
4:
make 9351b446-aef9-48dd-81eb-5cd882063563 success...
9351b446-aef9-48dd-81eb-5cd882063563
false
5:
make fdfba2dd-54a6-4e68-8ab1-9e38422c517a success...
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
6:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
7:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
8:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
9:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
10:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
11:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
12:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
13:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
true
14:
fdfba2dd-54a6-4e68-8ab1-9e38422c517a
false

```

结论：

```
定义了一个随机返回Boolean的方法，如果返回True 就 归还对象到对象池，否则不归还，如果归还了，那么取对象就池子里面取，否则就进行创建.
```
