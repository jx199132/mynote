# 数组、List、Set、Map的序列化与发序列化这里就简短说明

## 数组

```
Gson gson = new Gson();
int[] ints = {1, 2, 3, 4, 5};
String[] strings = {"abc", "def", "ghi"};
```
### 序列化

```
gson.toJson(ints);     ==> prints [1,2,3,4,5]
gson.toJson(strings);  ==> prints ["abc", "def", "ghi"]
```

### 反序列化

```
int[] ints2 = gson.fromJson("[1,2,3,4,5]", int[].class); 
```

## List

```
Gson gson = new Gson();
List<Integer> ints = new ArrayList<Integer>();
ints.add(1);
ints.add(2);
ints.add(3);
ints.add(4);
ints.add(5);
```
### 序列化

```
String json = gson.toJson(ints);
System.out.println(json);
```
### 反序列化

```
Type collectionType = new TypeToken<List<Integer>>(){}.getType();
List<Integer> ints2 = gson.fromJson(json, collectionType);
for(int i : ints2) {
	System.out.println(i);
}

// java.lang.reflect.Type
// com.google.gson.reflect.TypeToken
```

## Set (同List一样)

## Map

### 序列化

```
HashMap<String, List<String>> employees = new HashMap<>();  
employees.put("A", Arrays.asList("Andreas", "Arnold", "Aden"));  
employees.put("C", Arrays.asList("Christian", "Carter"));  
employees.put("M", Arrays.asList("Marcus", "Mary")); 
Gson gson = new Gson();  
String employeeJson = gson.toJson(employees);  
System.out.println(employeeJson);
```
结果
```
{
  "M": [
    "Marcus",
    "Mary"
  ],
  "C": [
    "Christian",
    "Carter"
  ],
  "A": [
    "Andreas",
    "Arnold",
    "Aden"
  ]
}
```


### 反序列化
需求 - 把json反序列化到 AmountWithCurrency 对象中

```
{
  "1$": {
    "amount": 1,
    "currency": "Dollar"
  },
  "2$": {
    "amount": 2,
    "currency": "Dollar"
  },
  "3€": {
    "amount": 3,
    "currency": "Euro"
  }
}

public class AmountWithCurrency {  
    String currency;
    int amount;
}
```

测试代码

```
public class AmountWithCurrency {  
    String currency;
    int amount;
}

String dollarJson = "{ '1$': { 'amount': 1, 'currency': 'Dollar'}, '2$': { 'amount': 2, 'currency': 'Dollar'}, '3€': { 'amount': 3, 'currency': 'Euro'} }";

Gson gson = new Gson();

Type amountCurrencyType =  
    new TypeToken<HashMap<String, AmountWithCurrency>>(){}.getType();

HashMap<String, AmountWithCurrency> amountCurrency =  
    gson.fromJson(dollarJson, amountCurrencyType);
```


