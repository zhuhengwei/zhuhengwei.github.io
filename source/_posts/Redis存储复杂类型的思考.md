---
title: Redis存储复杂类型的思考
date: 2016-01-27 22:51:05
categories: [Redis]
tags: [Redis,复杂类型]
---
### 关于Redis
　　Redis是一个开源的Key-Value内存数据库，redis会周期性的把更新的数据持久化写入磁盘，因此可以用作数据库、缓存和消息中间件。它支持多种类型的数据结构，如字符串（String）,列表（List），集合（Set）与范围查询，bitmaps，hyperloglogs和 地理空间（geospatial） 索引半径查询。
Jedis 是 Redis 官方首选的 Java 客户端开发包。使用方法非常简单。
<!-- more -->
```java
	import redis.clients.jedis.*

	Jedis jedis = new Jedis("localhost");
	jedis.set("foo", "bar");
	String value = jedis.get("foo");
```

### Redis存储list

#### jedis.rpush或者lpush
　　jedis存储list的常规方法，rpush和lpush区别在于LPUSH把新插入的值放在List的第一个，而rpush放在最后。官方解释：
> **Add the string value to the head (LPUSH) or tail (RPUSH) of the list stored at key.**代码如下：

```java
	long start = System.currentTimeMillis();
	Jedis jedis = new Jedis("172.23.27.80", 22367);
	// 开始前，先移除所有的内容
	jedis.del("java");
	
	List<String> list = new ArrayList<>();
	for (int i = 0; i < 10000; i++) {
		list.add("spring" + i);
	}
	
	// 存到redis
	for (int i = 0; i < 10000; i++) {
		jedis.rpush("java", "spring" + i);
	}
	// 获取redis中key为java的list
	List<String> resultList = jedis.lrange("java", 0, -1);
	for (int i = 0; i < resultList.size(); i++) {
		System.out.println(resultList.get(i));
	}
	
	long end = System.currentTimeMillis();
	System.out.println("rpush-lrange消耗时间：" + (end - start) + "ms");
	
	jedis.close();
```
　　这种方法是每个request一个response,效率极低。测试结果约11s.
　　![](http://i.imgur.com/1V15EPm.png)

#### jedis之Pipline
　　对于类似这种批量的请求，Jedis提供了pipline方法用于批量执行。
```java
	long start = System.currentTimeMillis();
	Jedis jedis = new Jedis("172.23.27.80", 22367);
	// 开始前，先移除所有的内容
	jedis.del("java");
	Pipeline pl = jedis.pipelined();
	// 存到redis
	for (int i = 0; i < 10000; i++) {
		pl.rpush("java", "spring" + i);
	}
	Response<List<String>> lrange = pl.lrange("java", 0, -1);
	pl.close();// Pipeline 需要在get前关闭
	List<String> list2 = lrange.get();
	
	for (String string : list2) {
		System.out.println(string);
	}
	
	long end = System.currentTimeMillis();
	System.out.println("Pipline消耗时间：" + (end - start) + "ms");
	jedis.close();
```

　　速度提升显著,只用了不到0.5s
　　![](http://i.imgur.com/8Q7s1K1.png)

####  换个思路，存JSON
　　以上可以看到，jedis虽然提供了操作list的方法，但是使用不方便，而且虽然用了Pipline在存取有10000个元素的list时效率提升显著，然而易用性大打折扣。**换个思路，把复杂类型转为Json存到redis**,比如list/map/list&lt;map>->json，取出后再解析成对应的数据类型。
```java
	long start = System.currentTimeMillis();
	Jedis jedis = new Jedis("172.23.27.80", 22367);
	// 开始前，先移除所有的内容
	jedis.del("java");
	
	// 准备数据
	List<String> list = new ArrayList<>();
	for (int i = 0; i < 10000; i++) {
		list.add("spring" + i);
	}
	jedis.set("java", JSON.toJSONString(list));
	
	// 获取、解析数据
	String value = jedis.get("java");
	JSONArray parseArray = JSON.parseArray(value);
	for (Object object : parseArray) {
		System.out.println(object.toString());
	}
	
	long end = System.currentTimeMillis();
	System.out.println("list 2 json消耗时间：" + (end - start) + "ms");
	
	jedis.close();
```
　　转成json，增加了程序可读性，并且只需要存取一次，效率必然很高。只用了427ms,比pipline还要快些~
　　![](http://i.imgur.com/gAenZ13.png)

　　把复杂类型转为Json，这种方式使redis支持更复杂的数据类型，比如list&lt;map&lt;String,Object>>,这个object只是一般的简单对象，如String,Integer,Date等等，还要支持更复杂的类型，list&lt;map&lt;String,Person>>则需要map-entity之间有映射转换工具。下面给出一个list&lt;map&lt;String,Object>> 的例子。
#### list&lt;map> To Json
```java
	long start = System.currentTimeMillis();
	Jedis jedis = new Jedis("172.23.27.80", 22367);
	// 开始前，先移除所有的内容
	jedis.del("java");
	// 准备数据
	List<Map<String, Object>> list = new ArrayList<>();
	for (int i = 0; i < 10000; i++) {
		Map<String, Object> map = new HashMap<>();
		map.put("framework", "spring" + i);
		list.add(map);
	}
	jedis.set("java", JSON.toJSONString(list));
	
	// 获取、解析数据
	String value = jedis.get("java");
	// json to list<map<>> 写成工具方法
	List<Map<String, Object>> resultList = jsonToList(value);
	for (int i = 0; i < resultList.size(); i++) {
		Map<String, Object> map2 = resultList.get(i);
		System.out.println(map2.get("framework"));
	}
	
	long end = System.currentTimeMillis();
	System.out.println("list<map> 2 json消耗时间：" + (end - start) + "ms");
	jedis.close();

```
存取有10000个map的list测试结果789ms,速率很好。

![](http://i.imgur.com/D274Lll.png)

以上示例中Json工具包是fastjson,Json转为list&lt;map> 需要用到下面的方法，写成工具方法备用。
```java
	public static Map<String, Object> jsonToMap(String json) {
		JSONObject jsonObject = JSONObject.parseObject(json);
		Map<String, Object> map = new HashMap<>();
		Set<String> keys = jsonObject.keySet();
		for (String key : keys) {
			map.put(key, jsonObject.get(key));
		}
		return map;
	}
	
	public static List<Map<String, Object>> jsonToList(String jsonList) {
		List<Map<String, Object>> list = new ArrayList<>();
		JSONArray parseArray = JSONArray.parseArray(jsonList);
		for (Object object : parseArray) {
			String jsonString = JSON.toJSONString(object);
			Map<String, Object> map = jsonToMap(jsonString);
			list.add(map);
		}
		return list;
	}
```

### 结论
　　**redis存取复杂类型时，将复杂类型转为json存储有着显著的效率提升和易操作性。把复杂类型转为Json存到redis，取出后再解析成对应的数据类型。这种方法可解决大部分的对象类型存储问题，适用于redis/memcache等K-V存储系统。**