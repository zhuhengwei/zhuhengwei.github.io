---
title: Java自定义比较器实现中文排序
date: 2016-02-27 20:55:45
categories: [Java]
tags: [比较器,中文]
---
### compareTo 方法
　　compareTo()是两个字符串对象比较大小，返回一个整数值，如果调用字符串对象大，返回正整数，反之，返回负整数。相等则返回0。compareTo()是两个字符串对象按ASCII比较大小（汉字是Unicode），返回一个整数值，如果调用字符串对象大，返回正整数，反之，返回负整数。相等则返回0。

<!-- more -->

### Comparator 比较器 ### 
　　Java 内实现自定义比较器比较简单，实现Comparator<T>接口的compare()这个方法来制定排序规则,按照Java规范应满足以下约定，否则会抛Comparison method violates its general contract 异常。规则如下：
![](http://i.imgur.com/sLl1FRR.png)

同时应满足以下约定：
+ 自反性 sgn(compare(x, y)) == -sgn(compare(y, x))
+ 传递性 compare(x, y) > 0 compare(y, z)>0) =>得出 compare(x, z)>0
+ 一致性 (compare(x, y)==0) == (x.equals(y))，这点规范中原文是“not strictly required”，不是必须的，但是实现者应该知道不一致的后果，所以尽量实现这一要求.

```java
	Comparator<String> comparator = new Comparator<String>() {
		@Override
		public int compare(String s1, String s2) {
			return s1.compareTo(s2);
		}
	};
```

以下代码示例：
```java
	@Test
	public void testCompare() {
		List<String> list = new ArrayList<>();
		list.add("java");
		list.add("php");
		list.add("c++");
		System.out.println("排序前-->" + list);
	
		Comparator<String> comparator = new Comparator<String>() {
			@Override
			public int compare(String s1, String s2) {
				return s1.compareTo(s2);
			}
		};
	
		Collections.sort(list, comparator);
	
		System.out.println("排序后-->" + list);
		Collections.reverse(list);
		System.out.println("排序后逆序-->" + list);
	}
```

输出结果为：
![](http://i.imgur.com/EHtWR95.png)


### Comparator中文排序
　　中文汉字是Unicode编码，所以排序时不是我们习惯用的拼音字母。如果还是刚才的实现，代码如下：
```java
	@Test
	public void testCompareCN() {
		List<String> list = new ArrayList<>();
		list.add("中国");// 中->20013 unicode编码的4E2D
		list.add("英国");// 英-->33521 unicode编码的82F1
		list.add("美国");// 美->32654 unicode编码的7F8E
		// 汉字unicode编码表 http://www.chi2ko.com/tool/CJK.htm
		System.out.println("排序前-->" + list);

		Comparator<String> comparator = new Comparator<String>() {
			@Override
			public int compare(String s1, String s2) {
				int b = s1.compareTo(s2);
				return b;
			}
		};

		Collections.sort(list, comparator);

		System.out.println("排序后-->" + list);
		Collections.reverse(list);
		System.out.println("排序后逆序-->" + list);

		// 输出字符编码对应的十进制
		//char a = '美';
		//System.out.println((int) a);
	}

```

输出结果为：
![](http://i.imgur.com/fQ1wLR9.png)

　　这个结果不符合我们的排序习惯，因此应该用Collator指定Locale.CHINA，代码应如下：
```java
	@Test
	public void testCollator() {
		List<String> list = new ArrayList<>();
		list.add("中国");
		list.add("英国");
		list.add("美国");
		System.out.println("排序前-->" + list);
		Collections.sort(list, new Comparator<String>() {
			@Override
			public int compare(String s1, String s2) {
				String o1 = "";
				String o2 = "";
				if (s1 != null) {
					o1 = s1;
				}
				if (s2 != null) {
					o2 = s2;
				}
				Collator instance = Collator.getInstance(Locale.CHINA);
				return instance.compare(o1, o2);
			}
		});

		System.out.println("排序后-->" + list);
		Collections.reverse(list);
		System.out.println("排序后逆序-->" + list);

	}
```

输出结果为：

![](http://i.imgur.com/qnAfDO5.png)

<font color=red>值得注意</font>的是，compareTo不能传入null，自定义比较器时要注意。