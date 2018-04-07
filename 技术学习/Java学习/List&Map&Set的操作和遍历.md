# List&Map&Set的操作和遍历

Java的三大集合即：Set、List、Map。

- Set：代表无序、不可重复的集合，常用的有HashSet（哈希表实现）、TreeSet（红黑树实现）；
- List：代表有序、可以重复的集合，比较常用的有ArrayList（数组实现）、LinkedList（链表实现）；
- Map：代表具有映射关系的集合，常用的有HashMap（哈希表实现）、TreeMap（红黑树实现）；

Java5以后又增加了Queue体系集合，代表一种队列集合实现，这里先不介绍。

List的实现类原理比较简单，Map比较复杂，而Set其实是基于Map的一种实现。

下面从各个集合的基本操作介绍一下，分别选取HashSet、ArrayList、HashMap三个典型的实现类：

### 1. HashSet

```java 
/** 
 * HashSet的增删遍历
 * @author wangjun 
 * @email  scuwangjun@hotmail.com
 * @time   2018年4月6日 下午2:40:33 
 */
public class HashSetOperation {

	public static void main(String[] args) {
		//初始化
		HashSet<String> set = new HashSet<>();
		//增
		set.add("key1");
		set.add("key2");
		set.add("key3");
		//删
		set.remove("key1");
		//遍历1
		//使用set.descendingIterator()方法可以反向遍历
		System.out.println("HashSet遍历1,使用Iterator:");
		Iterator<String> it = set.iterator();
		while(it.hasNext()) {
			System.out.println(it.next());
		}
		//遍历2
		System.out.println("HashSet遍历2,使用for：");
		for(String str: set) {
			System.out.println(str);
		}
	}
```

运行结果：

```shell
HashSet遍历1,使用Iterator:
key2
key3
HashSet遍历2,使用for：
key2
key3
```

### 2.ArrayList

```java
/** 
 * ArrayList的增删查改，遍历
 * @author wangjun 
 * @email  scuwangjun@hotmail.com
 * @time   2018年4月6日 下午2:25:43 
 */
public class ArrayListOperation {

	public static void main(String[] args) {
		//初始化
		List<String> list = new ArrayList<>();
		//增
		list.add("str1");
		list.add("str2");
		list.add("str3");
		//删
		list.remove(1);
		//查
		System.out.println("list的第二个元素是：" + list.get(1));
		//改
		list.set(0, "str11");
		System.out.println("最终的list：" + list.toString());
		//遍历1,使用for
		System.out.println("LinkedList遍历1,使用for:");
		for (int i = 0; i < list.size(); i++) {
			System.out.println(list.get(i));
		}
		//遍历2,使用增强for
		System.out.println("LinkedList遍历1,使用增强for:");
		for(String str: list) {
			System.out.println(str);
		}
		//遍历3，使用Iterator，集合类的通用遍历方式
		System.out.println("LinkedList遍历3，使用Iterator:");
		Iterator<String> it = list.iterator();
		while(it.hasNext()) {
			System.out.println(it.next());
		}
	}

}
```

运行结果：

```
list的第二个元素是：str3
最终的list：[str11, str3]
LinkedList遍历1,使用for:
str11
str3
LinkedList遍历1,使用增强for:
str11
str3
LinkedList遍历3，使用Iterator:
str11
str3
```

### 3.HashMap

```java
/** 
 * hashMap的增删查改
 * 无序
 * key相当于set，不可重复
 * value相当于list，可重复
 * @author wangjun 
 * @email  scuwangjun@hotmail.com
 * @time   2018年4月6日 下午2:30:31 
 */
public class HashMapOperation {

	public static void main(String[] args) {
		//初始化
		HashMap<String,String> map = new HashMap<>();
		//增
		map.put("key1", "value1");
		map.put("key2", "value2");
		map.put("key3", "value3");
		//删
		map.remove("key2");
		//查
		System.out.println("key1对应的valve为：" + map.get("key1"));
		//改
		map.replace("key3", "value33");
		System.out.println("最终的map是：" + map.toString());
		//遍历1，取出map中所有的key组成一个set
		System.out.println("HashMap遍历1，取出map中所有的key组成一个set:");
		for(String key: map.keySet()) {
			System.out.println("key:" + key + ",value:" + map.get(key));
		}
		//遍历2,取出key组成set后，通过Iterator遍历key
		System.out.println("HashMap遍历2,取出key组成set后，通过Iterator遍历key：");
		Iterator<String> it = map.keySet().iterator();
		while(it.hasNext()) {
			String key = it.next();
			String value = map.get(key);
			System.out.println("key:" + key + ",value:" + value);
		}
		//遍历3，取出map中实际存储的数据结构--Map.Entry,在HashMap中使用的是Node静态内部类
		//推荐这种，尤其是数据很大时
		System.out.println("HashMap遍历3，通过Map.Entry：");
		Set<Map.Entry<String, String>> entry = map.entrySet();
		for(Map.Entry<String, String> entryItem: entry) {
			String key = entryItem.getKey();
			String value = entryItem.getValue();
			System.out.println("key:" + key + ",value:" + value);
		}
		//遍历4,只能遍历value，不能遍历key,相当于取出map中左右的value组成一个list
		System.out.println("HashMap遍历4，只遍历value：");
		for(String value: map.values()) {
			System.out.println("value:" + value);
		}
	}

}
```

运行结果：

```
key1对应的valve为：value1
最终的map是：{key1=value1, key3=value33}
HashMap遍历1，取出map中所有的key组成一个set:
key:key1,value:value1
key:key3,value:value33
HashMap遍历2,取出key组成set后，通过Iterator遍历key：
key:key1,value:value1
key:key3,value:value33
HashMap遍历3，通过Map.Entry：
key:key1,value:value1
key:key3,value:value33
HashMap遍历4，只遍历value：
value:value1
value:value33
```

可以看到：

遍历Set一般常用2种方式；

遍历List一般常用3种方式；

遍历Map一般常用4种方式；

根据使用场景，选择合适的遍历方式。



