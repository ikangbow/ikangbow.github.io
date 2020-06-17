---
title: JAVA的map
date: 2020-06-08 11:45:45
tags: java
category: java
---
# Map的用法

## 类型介绍


- HashMap

	最常用的Map,它根据键的HashCode值存储数据，根据键可以直接获取它的值，具有很快的访问速度。HashMap最多只允许一条记录的键为null（多条会覆盖）；允许多条记录的值为null。非同步的。
- TreeMap
     
	能够把它保存的记录根据键（key）排序，默认是按升序排序，也可指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。TreeMap不允许key的值为null。非同步的。
- Hashtable

	与HashMap类似，不同的是：key和value的值均不允许为null;它支持线程同步，即任一时刻只有一个线程能写HashTable,因此也导致了HashTable在写入时会比较慢。
- LinkedHashMap

	保存了记录的插入顺序，在使用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的，在遍历的时候会比HashMap慢。key和value均允许为空。非同步的。

## Map用法

	用法
	Map<String,String> map = new HashMap<String,String>();

	插入元素

	map.put("key1","value1");

	获取元素

	map.get("key1");

	移除元素

	map.remove("key1")

## 四种常用Map插入与读取性能比较

![](批注 2020-06-08 124807.png)

	测试代码

	package net.xsoftlab.baike;
	import java.util.HashMap;
	import java.util.Hashtable;
	import java.util.LinkedHashMap;
	import java.util.Map;
	import java.util.Random;
	import java.util.TreeMap;
	import java.util.UUID;
	public class Test {
	    static int hashMapW = 0;
	    static int hashMapR = 0;
	    static int linkMapW = 0;
	    static int linkMapR = 0;
	    static int treeMapW = 0;
	    static int treeMapR = 0;
	    static int hashTableW = 0;
	    static int hashTableR = 0;
	    public static void main(String[] args) {
	        for (int i = 0; i < 10; i++) {
	            Test test = new Test();
	            test.test(100 * 10000);
	            System.out.println();
	        }
	        System.out.println("hashMapW = " + hashMapW / 10);
	        System.out.println("hashMapR = " + hashMapR / 10);
	        System.out.println("linkMapW = " + linkMapW / 10);
	        System.out.println("linkMapR = " + linkMapR / 10);
	        System.out.println("treeMapW = " + treeMapW / 10);
	        System.out.println("treeMapR = " + treeMapR / 10);
	        System.out.println("hashTableW = " + hashTableW / 10);
	        System.out.println("hashTableR = " + hashTableR / 10);
	    }
	    public void test(int size) {
	        int index;
	        Random random = new Random();
	        String[] key = new String[size];
	        // HashMap 插入
	        Map<String, String> map = new HashMap<String, String>();
	        long start = System.currentTimeMillis();
	        for (int i = 0; i < size; i++) {
	            key[i] = UUID.randomUUID().toString();
	            map.put(key[i], UUID.randomUUID().toString());
	        }
	        long end = System.currentTimeMillis();
	        hashMapW += (end - start);
	        System.out.println("HashMap插入耗时 = " + (end - start) + " ms");
	        // HashMap 读取
	        start = System.currentTimeMillis();
	        for (int i = 0; i < size; i++) {
	            index = random.nextInt(size);
	            map.get(key[index]);
	        }
	        end = System.currentTimeMillis();
	        hashMapR += (end - start);
	        System.out.println("HashMap读取耗时 = " + (end - start) + " ms");
	        // LinkedHashMap 插入
	        map = new LinkedHashMap<String, String>();
	        start = System.currentTimeMillis();
	        for (int i = 0; i < size; i++) {
	            key[i] = UUID.randomUUID().toString();
	            map.put(key[i], UUID.randomUUID().toString());
	        }
	        end = System.currentTimeMillis();
	        linkMapW += (end - start);
	        System.out.println("LinkedHashMap插入耗时 = " + (end - start) + " ms");
	        // LinkedHashMap 读取
	        start = System.currentTimeMillis();
	        for (int i = 0; i < size; i++) {
	            index = random.nextInt(size);
	            map.get(key[index]);
	        }
	        end = System.currentTimeMillis();
	        linkMapR += (end - start);
	        System.out.println("LinkedHashMap读取耗时 = " + (end - start) + " ms");
	        // TreeMap 插入
	        key = new String[size];
	        map = new TreeMap<String, String>();
	        start = System.currentTimeMillis();
	        for (int i = 0; i < size; i++) {
	            key[i] = UUID.randomUUID().toString();
	            map.put(key[i], UUID.randomUUID().toString());
	        }
	        end = System.currentTimeMillis();
	        treeMapW += (end - start);
	        System.out.println("TreeMap插入耗时 = " + (end - start) + " ms");
	        // TreeMap 读取
	        start = System.currentTimeMillis();
	        for (int i = 0; i < size; i++) {
	            index = random.nextInt(size);
	            map.get(key[index]);
	        }
	        end = System.currentTimeMillis();
	        treeMapR += (end - start);
	        System.out.println("TreeMap读取耗时 = " + (end - start) + " ms");
	        // Hashtable 插入
	        key = new String[size];
	        map = new Hashtable<String, String>();
	        start = System.currentTimeMillis();
	        for (int i = 0; i < size; i++) {
	            key[i] = UUID.randomUUID().toString();
	            map.put(key[i], UUID.randomUUID().toString());
	        }
	        end = System.currentTimeMillis();
	        hashTableW += (end - start);
	        System.out.println("Hashtable插入耗时 = " + (end - start) + " ms");
	        // Hashtable 读取
	        start = System.currentTimeMillis();
	        for (int i = 0; i < size; i++) {
	            index = random.nextInt(size);
	            map.get(key[index]);
	        }
	        end = System.currentTimeMillis();
	        hashTableR += (end - start);
	        System.out.println("Hashtable读取耗时 = " + (end - start) + " ms");
	    }
	}

## Map遍历

### 初始化数据

	Map<String,String> map = new HashMap<String,String>();
	map.put("key1","value1");
	map.put("key2","value2");

### 增强for循环遍历

使用keySet()遍历

	for(String key:map.keySet()){
		System.out.println(key+":"+map.get(key))
	}

使用entrySet()遍历

	for(Map.Entry<String,String> entry:map.entrySet()){
		System.out.println(entry.getKey()+":"+entry.getValue());
	}

### 迭代器遍历

使用keySet()遍历

	Iterator<String> iterator = map.keySet().iterator();
	while(iterator.hasNext()){
		String key = iterator.next();
		System.out.println(key+":"+map.get(key));
	}

使用entrySet()遍历

	Iterator<Map.Entry<String,String>> iterator = map.entrySet().iterator();
	while(iterator.hasNext()){
		Map.Entry<String,String> entry = iterator.next();
		System.out.println(entry.getKey()+":"+entry.getValue());
	}

## HashMap四种遍历方式性能比较

	package net.xsoftlab.baike;
	import java.util.HashMap;
	import java.util.Iterator;
	import java.util.Map;
	import java.util.Map.Entry;
	public class TestMap {
	    public static void main(String[] args) {
	        // 初始化，10W次赋值
	        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
	        for (int i = 0; i < 100000; i++)
	            map.put(i, i);
	        /** 增强for循环，keySet迭代 **/
	        long start = System.currentTimeMillis();
	        for (Integer key : map.keySet()) {
	            map.get(key);
	        }
	        long end = System.currentTimeMillis();
	        System.out.println("增强for循环，keySet迭代 -> " + (end - start) + " ms");
	        /** 增强for循环，entrySet迭代 */
	        start = System.currentTimeMillis();
	        for (Entry<Integer, Integer> entry : map.entrySet()) {
	            entry.getKey();
	            entry.getValue();
	        }
	        end = System.currentTimeMillis();
	        System.out.println("增强for循环，entrySet迭代 -> " + (end - start) + " ms");
	        /** 迭代器，keySet迭代 **/
	        start = System.currentTimeMillis();
	        Iterator<Integer> iterator = map.keySet().iterator();
	        Integer key;
	        while (iterator.hasNext()) {
	            key = iterator.next();
	            map.get(key);
	        }
	        end = System.currentTimeMillis();
	        System.out.println("迭代器，keySet迭代 -> " + (end - start) + " ms");
	        /** 迭代器，entrySet迭代 **/
	        start = System.currentTimeMillis();
	        Iterator<Map.Entry<Integer, Integer>> iterator1 = map.entrySet().iterator();
	        Map.Entry<Integer, Integer> entry;
	        while (iterator1.hasNext()) {
	            entry = iterator1.next();
	            entry.getKey();
	            entry.getValue();
	        }
	        end = System.currentTimeMillis();
	        System.out.println("迭代器，entrySet迭代 -> " + (end - start) + " ms");
	    }
	}

	运行三次，比较结果 第一次
	增强for循环，keySet迭代 -> 37 ms
	增强for循环，entrySet迭代 -> 19 ms
	迭代器，keySet迭代 -> 14 ms
	迭代器，entrySet迭代 -> 9 ms

	增强for循环，keySet迭代 -> 29 ms
	增强for循环，entrySet迭代 -> 22 ms
	迭代器，keySet迭代 -> 19 ms
	迭代器，entrySet迭代 -> 12 ms

	增强for循环，keySet迭代 -> 27 ms
	增强for循环，entrySet迭代 -> 19 ms
	迭代器，keySet迭代 -> 18 ms
	迭代器，entrySet迭代 -> 10 ms

总结：
1. 增强for循环使用方便，但性能较差，不适合处理超大量级的数据。
2. 迭代器的遍历速度要比增强for循环快很多，是增强for循环的2倍左右。
3. 使用entrySet遍历的速度比keySet快很多，是keySet的1.5倍左右。

## Map排序

HashMap、HashTable、LinkedHashMap排序

### HashMap

	Map<String,String> map = new HashMap<String,String>();
	map.put("b","b");
    map.put("a","c");
    map.put("c","a");
    //排序
    List<Map.Entry<String,String>> list = new ArrayList<Map.Entry<String, String>>(map.entrySet().size());
    Collections.sort(list, new Comparator<Map.Entry<String, String>>() {
        @Override
        public int compare(Map.Entry<String, String> o1, Map.Entry<String, String> o2) {
            return o1.getKey().compareTo(o2.getKey());
        }
    });

    for (Map.Entry<String,String> mapping:
    list) {
        System.out.println(mapping.getKey()+":"+mapping.getValue());
    }


### TreeMap

	Map<String, String> map = new TreeMap<String, String>(new Comparator<String>() {
	    @Override
	    public int compare(String o1, String o2) {
	        // 降序排序
	        return o1.compareTo(o2);
	    }
	});
	map.put("b", "b");
	map.put("a", "c");
	map.put("c", "a");
	for (String key : map.keySet()) {
	    System.out.println(key + " ：" + map.get(key));
	}

### 按value排序(通用)

	Map<String, String> map = new TreeMap<String, String>();
	map.put("b", "b");
	map.put("a", "c");
	map.put("c", "a");
	// 通过ArrayList构造函数把map.entrySet()转换成list
	List<Map.Entry<String, String>> list = new ArrayList<Map.Entry<String, String>>(map.entrySet());
	// 通过比较器实现比较排序
	Collections.sort(list, new Comparator<Map.Entry<String, String>>() {
	    @Override
	    public int compare(Map.Entry<String, String> mapping1, Map.Entry<String, String> mapping2) {
	        return mapping1.getValue().compareTo(mapping2.getValue());
	    }
	});
	for (String key : map.keySet()) {
	    System.out.println(key + " ：" + map.get(key));
	}


## 常用API

![](批注 2020-06-08 141629.png)

## 扩展List如何一边遍历一边删除

    public static void main(String[] args) {
	    List<String> platformList = new ArrayList<>();
	    platformList.add("博客园");
	    platformList.add("CSDN");
	    platformList.add("掘金");
	
	    for (String platform : platformList) {
	        if (platform.equals("博客园")) {
	            platformList.remove(platform);
	        }
	    }
	
	    System.out.println(platformList);
	}

	java.util.ConcurrentModificationException异常了，翻译成中文就是：并发修改异常

### 使用Iterator的remove()方法

    public static void main(String[] args) {
	    List<String> platformList = new ArrayList<>();
	    platformList.add("博客园");
	    platformList.add("CSDN");
	    platformList.add("掘金");
	
	    Iterator<String> iterator = platformList.iterator();
	    while (iterator.hasNext()) {
	        String platform = iterator.next();
	        if (platform.equals("博客园")) {
	            iterator.remove();
	        }
	    }
	
	    System.out.println(platformList);
	}

### 使用for循环正序遍历

    public static void main(String[] args) {
	    List<String> platformList = new ArrayList<>();
	    platformList.add("博客园");
	    platformList.add("CSDN");
	    platformList.add("掘金");
	
	    for (int i = 0; i < platformList.size(); i++) {
	        String item = platformList.get(i);
	
	        if (item.equals("博客园")) {
	            platformList.remove(i);
	            i = i - 1;
	        }
	    }
	
	    System.out.println(platformList);
	}

### 使用for循环倒序遍历

    public static void main(String[] args) {
	    List<String> platformList = new ArrayList<>();
	    platformList.add("博客园");
	    platformList.add("CSDN");
	    platformList.add("掘金");
	
	    for (int i = platformList.size() - 1; i >= 0; i--) {
	        String item = platformList.get(i);
	
	        if (item.equals("掘金")) {
	            platformList.remove(i);
	        }
	    }
	
	    System.out.println(platformList);
	}



