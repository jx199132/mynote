
## 序列化
- 准备工作（新建两个类）

```
public class UserAddress {  
    private String street;
    private String houseNumber;
    private String city;
    private String country;
    
    public UserAddress(String street, String houseNumber, String city, String country) {
		super();
		this.street = street;
		this.houseNumber = houseNumber;
		this.city = city;
		this.country = country;
	}
	public UserAddress() {
		super();
		// TODO Auto-generated constructor stub
	}
	@Override
	public String toString() {
		return "UserAddress [street=" + street + ", houseNumber=" + houseNumber + ", city=" + city + ", country="
				+ country + "]";
	}
	//== get  == set 省略
}



public class UserNested {  
    private String name;
    private String email;
    private boolean isDeveloper;
    private int age;
    
    // new, see below!
    private UserAddress userAddress;
    

	public UserNested() {
		super();
	}

	public UserNested(String name, String email, boolean isDeveloper, int age, UserAddress userAddress) {
		super();
		this.name = name;
		this.email = email;
		this.isDeveloper = isDeveloper;
		this.age = age;
		this.userAddress = userAddress;
	}
	
	@Override
	public String toString() {
		return "UserNested [name=" + name + ", email=" + email + ", isDeveloper=" + isDeveloper + ", age=" + age
				+ ", userAddress=" + userAddress + "]";
	}
	//== get  == set 省略
}
```
- 测试

```
    @Test
	public void test03() {
		UserAddress userAddress = new UserAddress("Main Street", "42A", "Magdeburg", "Germany");
		UserNested userObject = new UserNested("Norman", "norman@futurestud.io", true, 26, userAddress);
		Gson gson = new Gson();
		String userWithAddressJson = gson.toJson(userObject);
		System.out.println(userWithAddressJson);
	}
```

- 结果

```
{"name":"Norman","email":"norman@futurestud.io","isDeveloper":true,"age":26,"userAddress":{"street":"Main Street","houseNumber":"42A","city":"Magdeburg","country":"Germany"}}

```

## 反序列化
- 需求（有这样一段字符串需要转换为对象）

```
{
  "name": "Future Studio Steak House",
  "owner": {
    "name": "Christian",
    "address": {
      "city": "Magdeburg",
      "country": "Germany",
      "houseNumber": "42A",
      "street": "Main Street"
    }
  },
  "cook": {
    "age": 18,
    "name": "Marcus",
    "salary": 1500
  },
  "waiter": {
    "age": 18,
    "name": "Norman",
    "salary": 1000
  }
}
```

- 准备工作（新建Java类）

```
public class Waiter {  
    private String name;
    private int age;
    private int salary;
    
	public Waiter() {
		super();
	}

	public Waiter(String name, int age, int salary) {
		super();
		this.name = name;
		this.age = age;
		this.salary = salary;
	}
	
	@Override
	public String toString() {
		return "Waiter [name=" + name + ", age=" + age + ", salary=" + salary + "]";
	}
	//== get  == set 省略
}

public class Cook {  
    private String name;
    private int age;
    private int salary;
    
	public Cook(String name, int age, int salary) {
		super();
		this.name = name;
		this.age = age;
		this.salary = salary;
	}
	
	public Cook() {
		super();
		// TODO Auto-generated constructor stub
	}

	@Override
	public String toString() {
		return "Cook [name=" + name + ", age=" + age + ", salary=" + salary + "]";
	}
	//== get  == set 省略
}

public class Owner {  
    private String name;
    private UserAddress address;

	public Owner(String name, UserAddress address) {
		super();
		this.name = name;
		this.address = address;
	}

	public Owner() {
		super();
	}

	@Override
	public String toString() {
		return "Owner [name=" + name + ", address=" + address + "]";
	}
	//== get  == set 省略
}

public class Restaurant {  
    private String name;
    private Owner owner;
    private Cook cook;
    private Waiter waiter;
    
	public Restaurant(String name, Owner owner, Cook cook, Waiter waiter) {
		super();
		this.name = name;
		this.owner = owner;
		this.cook = cook;
		this.waiter = waiter;
	}
	
	public Restaurant() {
		super();
		// TODO Auto-generated constructor stub
	}
	
	@Override
	public String toString() {
		return "Restaurant [name=" + name + ", owner=" + owner + ", cook=" + cook + ", waiter=" + waiter + "]";
	}
	//== get  == set 省略
}
```

- 测试

```
    @Test
	public void test04() {
		String restaurantJson = "{ 'name':'Future Studio Steak House', 'owner':{ 'name':'Christian', 'address':{ 'city':'Magdeburg', 'country':'Germany', 'houseNumber':'42', 'street':'Main Street'}},'cook':{ 'age':18, 'name': 'Marcus', 'salary': 1500 }, 'waiter':{ 'age':18, 'name': 'Norman', 'salary': 1000}}";
		Gson gson = new Gson();
		Restaurant restaurantObject = gson.fromJson(restaurantJson, Restaurant.class); 
		System.out.println(restaurantObject);
	}
```

- 结果

```
Restaurant [name=Future Studio Steak House, owner=Owner [name=Christian, address=UserAddress [street=Main Street, houseNumber=42, city=Magdeburg, country=Germany]], cook=Cook [name=Marcus, age=18, salary=1500], waiter=Waiter [name=Norman, age=18, salary=1000]]

```
