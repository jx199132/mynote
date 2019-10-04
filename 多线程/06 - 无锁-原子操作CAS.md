# 悲观与乐观
对于并发而言，锁是一种悲观策略。总是假设每一次临界区操作都会产生冲突，因此，必须每次操作都小心翼翼。如果有多个线程访问临界区资源，就牺牲性能让线程进行等待，使得线程阻塞。而无锁是一种乐观的策略，它会假设对资源的访问时没有冲突的，不需要等待，所有线程都可用不停顿的执行。那么在遇到冲突的情况下采用一种叫做比较交换技术（CAS Compare And Swap）来鉴别线程冲突，一旦检查到冲突产生，就重试当前操作，直到没有冲突为止.

# Cas算法
cas算法 示例： 
在cas算法中包含三个参数 V，E，N 。 V表示要更新的变量，E表示预期值，N表示新值。仅当V等于E时，才会将V设置为N，如果V不等于E，说明其他线程做了更新，则当前线程什么都不做。最后cas返回当前V的真实值。CAS操作是抱着乐观的态度进行的，它总是认为自己的操作可以成功。当多个线程同时操作一个变量时，只会有一个会操作成功，其他的都会失败。 
而原子操作则是在cas的基础上，写入一个while（true) 在循环中，操作成功的线程就会跳出，而失败的线程在循环里面继续执行，直到成功操作为止。

# 原子整数(AtomicInteger)

```
import java.util.Random;
import java.util.concurrent.atomic.AtomicInteger;

public class M {
	
	private static AtomicInteger count = new AtomicInteger();
	
	private static int c = 0;
	
	static class T implements Runnable{

		@Override
		public void run() {
			for(int i = 0 ; i < 100 ; i++){
				try {
					Thread.sleep(new Random().nextInt(100));
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				count.addAndGet(1);
				c++;
			}
		}
		
	}
	
	public static void main(String[] args) {
		
		for(int i = 0 ; i < 5 ; i++){
			new Thread(new T()).start();
		}
		
		while(true){
			System.out.println(count.get());
			System.out.println("c : " + c);
		}
	}
	
}
```
输出结果 count 会非常稳定 最终等于 500   而 c  不稳定小于等于500 

与AtomicInteger类似的还有AtomicLong，AtomicBoolean分别表示long，boolean类型.

## 主要方法
```
public final int get()  //取得当前值
public final void set(int newValue) //设置当前值
public final int getAndSet(int newValue) //设置新值，并返回旧值
3
public final boolean compareAndSet(int expect, int u)
//如果当前值为expect，则设置为u
public final int getAndIncrement() //当前值加1，返回旧值
public final int getAndDecrement() //当前值减1，返回旧值
public final int getAndAdd(int delta) //当前值增加delta，返回旧值
public final int incrementAndGet() //当前值加1，返回新值
public final int decrementAndGet() //当前值减1，返回新值
public final int addAndGet(int delta) //当前值增加delta，返回新值
```

# 原子的对象引用(AtomicReference)
AtomicReference与AtomicInteger非常类似，不同之处在于AtomicInteger是对于整数的封装，而AtomicReference是对于普通对象的引用，也就是说它可以保证在修改对象的引用时线程的安全性。

```
public class AtomicReferenceTest {
	private static AtomicReference<String> ar = new AtomicReference<String>();
	
	public static void main(String[] args) {
		
		ar.set("abc");
		
		for (int i = 0 ; i < 10 ; i++){
			new Thread(new Runnable() {
				@Override
				public void run() {
					if(ar.compareAndSet("abc", "def")){
						System.out.println(Thread.currentThread().getId() + " 修改 字符串 abc  改成 def");
					}else{
						System.out.println(Thread.currentThread().getId() + " 没有进行修改");
					}
				}
			}).start();
			
		}
		
	}
}

```

## 主要方法
get()
set(V)
compareAndSe
getAndSet(V)

# 带时间梭的原子对象引用（AtomicStampedReference）
线程判断对象是否可以正取写入的条件时对象的当前值和期望值是否一致，一般情况下下都是没有问题的。但是存在特殊情况，比如：这个对象的值被其他线程改变了之后，又改了回去。进行了两次修改恢复到了旧的数值，这样的话当前线程是无法判断是否被更改过.而AtomicStampedReference可以解决这个问题.
## 主要方法
比较设置 参数依次为：期望值 写入新值 期望时间戳 新时间戳

public boolean compareAndSet(V expectedReference,V newReference,int expectedStamp,int newStamp) 

获得当前对象引用

public V getReference()

获得当前时间戳

public int getStamp()

设置当前对象引用和时间戳

public void set(V newReference, int newStamp)

# 数组也能无锁(AtomicIntegerArray)
除了基本数据类型之外，JDK还提供了数组等结构，AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray。分别表示整数原子数组，长整型原子数组，对象原子数组。
## 主要方法
获得数组第i个下标的元素

public final int get(int i)

获得数组的长度

public final int length()

将数组第i个下标设置为newValue，并返回旧的值

public final int getAndSet(int i, int newValue)

进行CAS操作，如果第i个下标的元素等于expect，则设置为update，设置成功返回true

public final boolean compareAndSet(int i, int expect, int update)

将第i个下标的元素加1

public final int getAndIncrement(int i)

将第i个下标的元素减1

public final int getAndDecrement(int i)

将第i个下标的元素增加delta（delta可以是负数）

public final int getAndAdd(int i, int delta)

# 普通变量也能原子操作
让普通变量也享受原子操作，有时候系统代码中数据类型固定了就是int类型，但是后面要加上原子操作，那么可以用AtomicIntegerFieldUpdater 来进行操作
AtomicIntegerFieldUpdater
AtomicLongFieldUpdater
AtomicReferenceFieldUpdater

