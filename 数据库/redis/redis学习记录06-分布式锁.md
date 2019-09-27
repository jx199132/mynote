# 使用场景
在很多互联网产品应用中，有些场景需要加锁处理，比如：秒杀.
# 命令
## SETNX
对一个键加锁，如果这个键存在那么返回0，不存在则返回1

```
SETNX(key,value)
```
# 示例-模拟购物
模拟1000个用户同时购买书籍，书的库存为100，秒杀抢购有效数量为20，最终会有20个用户买到书，书的库存最后改成80.
## 准备
在redis中加入一个key,设置值为100

```
set bookcount 100

```

## jedis代码

```
public class Lock {
	
	
	public static void main(String[] args) {
		
		for(int i = 0 ; i < 1000 ; i ++){
			new Thread(new User(String.valueOf(i+1))).start();
		}
		
	}
	
	static class User implements Runnable{
		private static int count = 0;		//计算器
		private static int total = 20;		//总共可以购买的个数
		private static long usetime = 10 * 1000;//预计购买到书的时间
		private Jedis jedis = new Jedis("192.168.95.128",6379,10000);
		private String name;
		public User(String name){
			this.name = name;
		}
		public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
		@Override
		public void run() {
			
			buyBook();
			
		}
		
		public boolean buyBook(){
			long curr = System.currentTimeMillis();
			long end = curr + usetime;
			while(System.currentTimeMillis() <= end){
				if(jedis.setnx("buybook", String.valueOf(System.currentTimeMillis())) == 1){
					if(count < total){
						jedis.decr("bookcount");
						count++;
						System.out.println(this.name + " 购买成功  !");
					}
					jedis.del("buybook");
					return true;
				}
				jedis.expire("buybook", 10);
				System.out.println(Thread.currentThread().getName() + " 购买失败");
				try {
					Thread.sleep(1);
				} catch (InterruptedException e) {
					Thread.currentThread().interrupt();
				}
			}
			return false;
		}
	}
}
```