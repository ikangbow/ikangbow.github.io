---
title: Redis相关及实践
date: 2018-12-15 10:17:10
tags: redis
categories: redis
---
# 定义
redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。
Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。 
Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。
redis的官网地址，非常好记，是redis.io。（特意查了一下，域名后缀io属于国家域名，是british Indian Ocean territory，即英属印度洋领地）
目前，Vmware在资助着redis项目的开发和维护。[详见百度百科](https://baike.baidu.com/item/Redis/6549233?fr=aladdin "百度百科")

---
# RedisTemplate常用集合-opsForValue
## 1.set(K key, V value)
	新增一个字符串类型的值,key是键，value是值。

    redisTemplate.opsForValue().set("stringValue","bbb"); 

## 2.get(Object key)
	获取key键对应的值

    String stringValue = redisTemplate.opsForValue().get("stringValue")+"";  
    System.out.println("通过get(Object key)方法获取set(K key, V value)方法新增的字符串值:" + stringValue); 


## 3.append(K key, String value)
	在原有的值基础上新增字符串到末尾。

    redisTemplate.opsForValue().append("stringValue","aaa"); 
    String stringValueAppend = redisTemplate.opsForValue().get("stringValue")+""; `
    System.out.println("通过append(K key, String value)方法修改后的字符串:"+stringValueAppend);  

## 4.get(K key, long start, long end)
	截取key键对应值得字符串，从开始下标位置开始到结束下标的位置(包含结束下标)的字符串。
    
    String cutString = redisTemplate.opsForValue().get("stringValue",0,3);
    System.out.println("通过get(K key, long start, long end)方法获取截取的字符串:"+cutString);


## 5.getAndSet(K key, V value)
	获取原来key键对应的值并重新赋新值。

    String oldAndNewStringValue = redisTemplate.opsForValue().getAndSet("stringValue","ccc")+"";
    System.out.print("通过getAndSet(K key, V value)方法获取原来的" + oldAndNewStringValue + ",");  
    String newStringValue = redisTemplate.opsForValue().get("stringValue")+"";
    System.out.println("修改过后的值:"+newStringValue); 

## 6.setBit(K key, long offset, boolean value)
	key键对应的值value对应的ascii码,在offset的位置(从左向右数)变为value。

    redisTemplate.opsForValue().setBit("stringValue",1,false);
    newStringValue = redisTemplate.opsForValue().get("stringValue")+"";
    System.out.println("通过setBit(K key,long offset,boolean value)方法修改过后的值:"+newStringValue); 


## 7.getBit(K key, long offset)
	判断指定的位置ASCII码的bit位是否为1。

     boolean bitBoolean = redisTemplate.opsForValue().getBit("stringValue",1);
     boolean bitBoolean = redisTemplate.opsForValue().getBit("stringValue",1);


## 8.size(K key)
	获取指定字符串的长度。

    Long stringValueLength = redisTemplate.opsForValue().size("stringValue");
    Long stringValueLength = redisTemplate.opsForValue().size("stringValue");


## 9.increment(K key, double delta)
	 以增量的方式将double值存储在变量中。

    double stringValueDouble = redisTemplate.opsForValue().increment("doubleValue",5);
    System.out.println("通过increment(K key, double delta)方法以增量方式存储double值:" + stringValueDouble);

## 10.increment(K key, long delta)
	以增量的方式将long值存储在变量中。

    double stringValueLong = redisTemplate.opsForValue().increment("longValue",6); System.out.println("通过increment(K key, long delta)方法以增量方式存储long值:" + stringValueLong);

## 11.setIfAbsent(K key, V value)
	如果键不存在则新增,存在则不改变已经有的值。

    boolean absentBoolean = redisTemplate.opsForValue().setIfAbsent("absentValue","fff");  
    System.out.println("通过setIfAbsent(K key, V value)方法判断变量值absentValue是否存在:" + absentBoolean);  
    if(absentBoolean){  
    String absentValue = redisTemplate.opsForValue().get("absentValue")+"";  
    System.out.print(",不存在，则新增后的值是:"+absentValue);  
    boolean existBoolean = redisTemplate.opsForValue().setIfAbsent("absentValue","eee");  
    System.out.print(",再次调用setIfAbsent(K key, V value)判断absentValue是否存在并重新赋值:" + existBoolean);  
		if(!existBoolean){  
		    absentValue = redisTemplate.opsForValue().get("absentValue")+"";  
		    System.out.print("如果存在,则重新赋值后的absentValue变量的值是:" + absentValue);  
		}  
    }
## 12.set(K key, V value, long timeout, TimeUnit unit)
	设置变量值的过期时间。
	
    redisTemplate.opsForValue().set("timeOutValue","timeOut",5,TimeUnit.SECONDS);  
    String timeOutValue = redisTemplate.opsForValue().get("timeOutValue")+"";  
    System.out.println("通过set(K key, V value, long timeout, TimeUnit unit)方法设置过期时间，过期之前获取的数据:"+timeOutValue);  
    Thread.sleep(5*1000);  
    timeOutValue = redisTemplate.opsForValue().get("timeOutValue")+"";  
    System.out.print(",等待10s过后，获取的值:"+timeOutValue);  

## 13.set(K key, V value, long offset)
	 覆盖从指定位置开始的值。

    redisTemplate.opsForValue().set("absentValue","dd",1);  
    String overrideString = redisTemplate.opsForValue().get("absentValue")+"";  
    System.out.println("通过set(K key, V value, long offset)方法覆盖部分的值:"+overrideString);

## 14.multiSet(Map<? extends K,? extends V> map)
	设置map集合到redis。

    Map valueMap = new HashMap();  
    valueMap.put("valueMap1","map1");  
    valueMap.put("valueMap2","map2");  
    valueMap.put("valueMap3","map3");  
    redisTemplate.opsForValue().multiSet(valueMap);  

## 15.multiGet(Collection<K> keys)
	 根据集合取出对应的value值。

    //根据List集合取出对应的value值  
    List paraList = new ArrayList();  
    paraList.add("valueMap1");  
    paraList.add("valueMap2");  
    paraList.add("valueMap3");  
    List<String> valueList = redisTemplate.opsForValue().multiGet(paraList);  
    for (String value : valueList){  
		System.out.println("通过multiGet(Collection<K> keys)方法获取map值:" + value);  
    } 

## 16.multiSetIfAbsent(Map<? extends K,? extends V> map)
	如果对应的map集合名称不存在，则添加，如果存在则不做修改。

    Map valueMap = new HashMap();  
    valueMap.put("valueMap1","map1");  
    valueMap.put("valueMap2","map2");  
    valueMap.put("valueMap3","map3");  
    redisTemplate.opsForValue().multiSetIfAbsent(valueMap);  

---

# RedisTemplate常用集合-opsForList

## 1.leftPush(K key, V value)
	在变量左边添加元素值。

    redisTemplate.opsForList().leftPush("list","a");
    redisTemplate.opsForList().leftPush("list","b");
    redisTemplate.opsForList().leftPush("list","c");
    
## 2.index(K key, long index)
	获取集合指定位置的值。

    String listValue = redisTemplate.opsForList().index("list",1) + "";
    System.out.println("通过index(K key, long index)方法获取指定位置的值:" + listValue);

## 3.range(K key, long start, long end)
	获取指定区间的值。

    List<Object> list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过range(K key, long start, long end)方法获取指定范围的集合值:"+list);

## 4.leftPush(K key, V pivot, V value)
	把最后一个参数值放到指定集合的第一个出现中间参数的前面，如果中间参数值存在的话。

    redisTemplate.opsForList().leftPush("list","a","n");
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过leftPush(K key, V pivot, V value)方法把值放到指定参数值前面:" + list);

## 5.leftPushAll(K key, V... values)
	向左边批量添加参数元素。

    redisTemplate.opsForList().leftPushAll("list","w","x","y");
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过leftPushAll(K key, V... values)方法批量添加元素:" + list);
    
## 6.leftPushAll(K key, Collection<V> values)
	以集合的方式向左边批量添加元素。

    List newList = new ArrayList();
    newList.add("o");
    newList.add("p");
    newList.add("q");
    redisTemplate.opsForList().leftPushAll("list",newList);
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过leftPushAll(K key, Collection<V> values)方法以集合的方式批量添加元素:" + list);

## 7.leftPushIfPresent(K key, V value)
	 如果存在集合则添加元素。

    redisTemplate.opsForList().leftPushIfPresent("presentList","o");
    list =  redisTemplate.opsForList().range("presentList",0,-1);
    System.out.println("通过leftPushIfPresent(K key, V value)方法向已存在的集合添加元素:" + list);

## 8.rightPush(K key, V value)
	向集合最右边添加元素。

    redisTemplate.opsForList().rightPush("list","w");
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过rightPush(K key, V value)方法向最右边添加元素:" + list);

## 9.rightPush(K key, V pivot, V value)
	向集合中第一次出现第二个参数变量元素的右边添加第三个参数变量的元素值。

    redisTemplate.opsForList().rightPush("list","w","r");
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过rightPush(K key, V pivot, V value)方法向最右边添加元素:" + list);

## 10.rightPushAll(K key, V... values)
	向右边批量添加元素。

    redisTemplate.opsForList().rightPushAll("list","j","k");
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过rightPushAll(K key, V... values)方法向最右边批量添加元素:" + list);

## 11.rightPushAll(K key, Collection<V> values)
	以集合方式向右边添加元素。

    newList.clear();
    newList.add("g");
    newList.add("h");
    redisTemplate.opsForList().rightPushAll("list",newList);
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过rightPushAll(K key, Collection<V> values)方法向最右边以集合方式批量添加元素:" + list);

## 12.rightPushIfPresent(K key, V value)
	向已存在的集合中添加元素。

    redisTemplate.opsForList().rightPushIfPresent("presentList","d");
    list =  redisTemplate.opsForList().range("presentList",0,-1);
    System.out.println("通过rightPushIfPresent(K key, V value)方法已存在的集合向最右边添加元素:" + list);

## 13.size(K key)
	获取集合长度。

    long listLength = redisTemplate.opsForList().size("list");
    System.out.println("通过size(K key)方法获取集合list的长度为:" + listLength);

## 14.leftPop(K key)
	移除集合中的左边第一个元素。

    Object popValue = redisTemplate.opsForList().leftPop("list");
    System.out.print("通过leftPop(K key)方法移除的元素是:" + popValue);
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println(",剩余的元素是:" + list);

## 15.leftPop(K key, long timeout, TimeUnit unit)
	移除集合中左边的元素在等待的时间里，如果超过等待的时间仍没有元素则退出。

    popValue = redisTemplate.opsForList().leftPop("presentList",1, TimeUnit.SECONDS);
    System.out.print("通过leftPop(K key, long timeout, TimeUnit unit)方法移除的元素是:" + popValue);
    list =  redisTemplate.opsForList().range("presentList",0,-1);
    System.out.println(",剩余的元素是:" + list);

## 16.rightPop(K key)
	移除集合中右边的元素。
    popValue = redisTemplate.opsForList().rightPop("list");
    System.out.print("通过rightPop(K key)方法移除的元素是:" + popValue);
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println(",剩余的元素是:" + list);
    
## 17.rightPop(K key, long timeout, TimeUnit unit)
	移除集合中右边的元素在等待的时间里，如果超过等待的时间仍没有元素则退出。

    popValue = redisTemplate.opsForList().rightPop("presentList",1, TimeUnit.SECONDS);
    System.out.print("通过rightPop(K key, long timeout, TimeUnit unit)方法移除的元素是:" + popValue);
    list =  redisTemplate.opsForList().range("presentList",0,-1);
    System.out.println(",剩余的元素是:" + list);

## 18.rightPopAndLeftPush(K sourceKey, K destinationKey)
	移除集合中右边的元素，同时在左边加入一个元素。

    popValue = redisTemplate.opsForList().rightPopAndLeftPush("list","12");
    System.out.print("通过rightPopAndLeftPush(K sourceKey, K destinationKey)方法移除的元素是:" + popValue);
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println(",剩余的元素是:" + list);

## 19.rightPopAndLeftPush(K sourceKey, K destinationKey, long timeout, TimeUnit unit)
	 移除集合中右边的元素在等待的时间里，同时在左边添加元素，如果超过等待的时间仍没有元素则退出。

    popValue = redisTemplate.opsForList().rightPopAndLeftPush("presentList","13",1,TimeUnit.SECONDS);
    System.out.println("通过rightPopAndLeftPush(K sourceKey, K destinationKey, long timeout, TimeUnit unit)方法移除的元素是:" + popValue);
    list =  redisTemplate.opsForList().range("presentList",0,-1);
    System.out.print(",剩余的元素是:" + list);

## 20.set(K key, long index, V value)
	在集合的指定位置插入元素,如果指定位置已有元素，则覆盖，没有则新增，超过集合下标+n则会报错。

    redisTemplate.opsForList().set("presentList",3,"15");
    list =  redisTemplate.opsForList().range("presentList",0,-1);
    System.out.print("通过set(K key, long index, V value)方法在指定位置添加元素后:" + list);

## 21.remove(K key, long count, Object value)
	从存储在键中的列表中删除等于值的元素的第一个计数事件。count> 0：删除等于从左到右移动的值的第一个元素；count< 0：删除等于从右到左移动的值的第一个元素；count = 0：删除等于value的所有元素。

    long removeCount = redisTemplate.opsForList().remove("list",0,"w");
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过remove(K key, long count, Object value)方法移除元素数量:" + removeCount);
    System.out.println(",剩余的元素:" + list);

## 22.trim(K key, long start, long end)
	截取集合元素长度，保留长度内的数据。

    redisTemplate.opsForList().trim("list",0,5);
    list =  redisTemplate.opsForList().range("list",0,-1);
    System.out.println("通过trim(K key, long start, long end)方法截取后剩余元素:" + list);

---

# RedisTemplate常用集合-opsForHash

## 1.put(H key, HK hashKey, HV value)
	新增hashMap值。

    redisTemplate.opsForHash().put("hashValue","map1","map1-1");
    redisTemplate.opsForHash().put("hashValue","map2","map2-2");

## 2.values(H key)
	获取指定变量中的hashMap值。

    List<Object> hashList = redisTemplate.opsForHash().values("hashValue");
    System.out.println("通过values(H key)方法获取变量中的hashMap值:" + hashList);

## 3.entries(H key)
	获取变量中的键值对。

    Map<Object,Object> map = redisTemplate.opsForHash().entries("hashValue");
    System.out.println("通过entries(H key)方法获取变量中的键值对:" + map);

## 4.get(H key, Object hashKey)
	获取变量中的指定map键是否有值,如果存在该map键则获取值，没有则返回null。

    Object mapValue = redisTemplate.opsForHash().get("hashValue","map1");
    System.out.println("通过get(H key, Object hashKey)方法获取map键的值:" + mapValue);

## 5.hasKey(H key, Object hashKey)
	 判断变量中是否有指定的map键。

    boolean hashKeyBoolean = redisTemplate.opsForHash().hasKey("hashValue","map3");
    System.out.println("通过hasKey(H key, Object hashKey)方法判断变量中是否存在map键:" + hashKeyBoolean);

## 6.keys(H key)
	获取变量中的键。

    Set<Object> keySet = redisTemplate.opsForHash().keys("hashValue");
    System.out.println("通过keys(H key)方法获取变量中的键:" + keySet);

## 7.size(H key)
	获取变量的长度。

    long hashLength = redisTemplate.opsForHash().size("hashValue");
    System.out.println("通过size(H key)方法获取变量的长度:" + hashLength);

## 8.increment(H key, HK hashKey, double delta)
	使变量中的键以double值的大小进行自增长。

    double hashIncDouble = redisTemplate.opsForHash().increment("hashInc","map1",3);
    System.out.println("通过increment(H key, HK hashKey, double delta)方法使变量中的键以值的大小进行自增长:" + hashIncDouble);

## 9.increment(H key, HK hashKey, long delta)
	使变量中的键以long值的大小进行自增长。

    long hashIncLong = redisTemplate.opsForHash().increment("hashInc","map2",6);
    System.out.println("通过increment(H key, HK hashKey, long delta)方法使变量中的键以值的大小进行自增长:" + hashIncLong);

## 10.multiGet(H key, Collection<HK> hashKeys)
	以集合的方式获取变量中的值。

    List<Object> list = new ArrayList<Object>();
    list.add("map1");
    list.add("map2");
    List mapValueList = redisTemplate.opsForHash().multiGet("hashValue",list);
    System.out.println("通过multiGet(H key, Collection<HK> hashKeys)方法以集合的方式获取变量中的值:"+mapValueList);

## 11.putAll(H key, Map<? extends HK,? extends HV> m)
	以map集合的形式添加键值对。

    Map newMap = new HashMap();
    newMap.put("map3","map3-3");
    newMap.put("map5","map5-5");
    redisTemplate.opsForHash().putAll("hashValue",newMap);
    map = redisTemplate.opsForHash().entries("hashValue");
    System.out.println("通过putAll(H key, Map<? extends HK,? extends HV> m)方法以map集合的形式添加键值对:" + map);

## 12.putIfAbsent(H key, HK hashKey, HV value)
	如果变量值存在，在变量中可以添加不存在的的键值对，如果变量不存在，则新增一个变量，同时将键值对添加到该变量。

    redisTemplate.opsForHash().putIfAbsent("hashValue","map6","map6-6");
    map = redisTemplate.opsForHash().entries("hashValue");
    System.out.println("通过putIfAbsent(H key, HK hashKey, HV value)方法添加不存在于变量中的键值对:" + map);

## 13.scan(H key, ScanOptions options)
	匹配获取键值对，ScanOptions.NONE为获取全部键对，ScanOptions.scanOptions().match("map1").build()     匹配获取键位map1的键值对,不能模糊匹配。

    Cursor<Map.Entry<Object,Object>> cursor = redisTemplate.opsForHash().scan("hashValue",ScanOptions.scanOptions().match("map1").build());
    //Cursor<Map.Entry<Object,Object>> cursor = redisTemplate.opsForHash().scan("hashValue",ScanOptions.NONE);
    while (cursor.hasNext()){
	    Map.Entry<Object,Object> entry = cursor.next();
	    System.out.println("通过scan(H key, ScanOptions options)方法获取匹配键值对:" + entry.getKey() + "---->" + entry.getValue());
    }

## 14.delete(H key, Object... hashKeys)
	删除变量中的键值对，可以传入多个参数，删除多个键值对。

    redisTemplate.opsForHash().delete("hashValue","map1","map2");
    map = redisTemplate.opsForHash().entries("hashValue");
    System.out.println("通过delete(H key, Object... hashKeys)方法删除变量中的键值对后剩余的:" + map);

---

# RedisTemplate常用集合-opsForSet

## 1.add(K key, V... values)
	向变量中批量添加值。

    redisTemplate.opsForSet().add("setValue","A","B","C","B","D","E","F");

## 2.members(K key)
	 获取变量中的值。

    Set set = redisTemplate.opsForSet().members("setValue");
    System.out.println("通过members(K key)方法获取变量中的元素值:" + set);

## 3.size(K key)
	获取变量中值的长度。

    long setLength = redisTemplate.opsForSet().size("setValue");
    System.out.println("通过size(K key)方法获取变量中元素值的长度:" + setLength);

## 4.randomMember(K key)
	随机获取变量中的元素。

    Object randomMember = redisTemplate.opsForSet().randomMember("setValue");
    System.out.println("通过randomMember(K key)方法随机获取变量中的元素:" + randomMember);

## 5.randomMembers(K key, long count)
	随机获取变量中指定个数的元素。

    List randomMembers = redisTemplate.opsForSet().randomMembers("setValue",2);
    System.out.println("通过randomMembers(K key, long count)方法随机获取变量中指定个数的元素:" + randomMembers);

## 6.isMember(K key, Object o)
	检查给定的元素是否在变量中。

    boolean isMember = redisTemplate.opsForSet().isMember("setValue","A");
    System.out.println("通过isMember(K key, Object o)方法检查给定的元素是否在变量中:" + isMember);

## 7.move(K key, V value, K destKey)
	转移变量的元素值到目的变量。

    boolean isMove = redisTemplate.opsForSet().move("setValue","A","destSetValue");
    if(isMove){
	    set = redisTemplate.opsForSet().members("setValue");
	    System.out.print("通过move(K key, V value, K destKey)方法转移变量的元素值到目的变量后的剩余元素:" + set);
	    set = redisTemplate.opsForSet().members("destSetValue");
	    System.out.println(",目的变量中的元素值:" + set);
    }

## 8.pop(K key)
	弹出变量中的元素。

    Object popValue = redisTemplate.opsForSet().pop("setValue");
    System.out.print("通过pop(K key)方法弹出变量中的元素:" + popValue);
    set = redisTemplate.opsForSet().members("setValue");
    System.out.println(",剩余元素:" + set)

## 9.remove(K key, Object... values)
	批量移除变量中的元素。

    long removeCount = redisTemplate.opsForSet().remove("setValue","E","F","G");
    System.out.print("通过remove(K key, Object... values)方法移除变量中的元素个数:" + removeCount);
    set = redisTemplate.opsForSet().members("setValue");
    System.out.println(",剩余元素:" + set);

## 10.scan(K key, ScanOptions options)
	匹配获取键值对，ScanOptions.NONE为获取全部键值对；ScanOptions.scanOptions().match("C").build()匹配获取键位map1的键值对,不能模糊匹配。

    //Cursor<Object> cursor = redisTemplate.opsForSet().scan("setValue", ScanOptions.NONE);
    Cursor<Object> cursor = redisTemplate.opsForSet().scan("setValue", ScanOptions.scanOptions().match("C").build());
    while (cursor.hasNext()){
	    Object object = cursor.next();
	    System.out.println("通过scan(K key, ScanOptions options)方法获取匹配的值:" + object);
    }

## 11.difference(K key, Collection<K> otherKeys)
	通过集合求差值。

    List list = new ArrayList();
    list.add("destSetValue");
    Set differenceSet = redisTemplate.opsForSet().difference("setValue",list);
    System.out.println("通过difference(K key, Collection<K> otherKeys)方法获取变量中与给定集合中变量不一样的值:" + differenceSet);

## 12.difference(K key, K otherKey)
	通过给定的key求2个set变量的差值。

    differenceSet = redisTemplate.opsForSet().difference("setValue","destSetValue");
    System.out.println("通过difference(K key, Collection<K> otherKeys)方法获取变量中与给定变量不一样的值:" + differenceSet);

## 13.differenceAndStore(K key, K otherKey, K destKey)
	将求出来的差值元素保存。

    redisTemplate.opsForSet().differenceAndStore("setValue","destSetValue","storeSetValue");
    set = redisTemplate.opsForSet().members("storeSetValue");
    System.out.println("通过differenceAndStore(K key, K otherKey, K destKey)方法将求出来的差值元素保存:" + set);

## 14.differenceAndStore(K key, Collection<K> otherKeys, K destKey)
	将求出来的差值元素保存。

    redisTemplate.opsForSet().differenceAndStore("setValue",list,"storeSetValue");
    set = redisTemplate.opsForSet().members("storeSetValue");
    System.out.println("通过differenceAndStore(K key, Collection<K> otherKeys, K destKey)方法将求出来的差值元素保存:" + set);

## 15.distinctRandomMembers(K key, long count)
	获取去重的随机元素。

    set = redisTemplate.opsForSet().distinctRandomMembers("setValue",2);
    System.out.println("通过distinctRandomMembers(K key, long count)方法获取去重的随机元素:" + set);

## 16.intersect(K key, K otherKey)
	获取2个变量中的交集。

    set = redisTemplate.opsForSet().intersect("setValue","destSetValue");
    System.out.println("通过intersect(K key, K otherKey)方法获取交集元素:" + set);

## 17.intersect(K key, Collection<K> otherKeys)
	获取多个变量之间的交集。

    set = redisTemplate.opsForSet().intersect("setValue",list);
    System.out.println("通过intersect(K key, Collection<K> otherKeys)方法获取交集元素:" + set);

## 18.intersectAndStore(K key, K otherKey, K destKey)
	获取2个变量交集后保存到最后一个参数上。

    redisTemplate.opsForSet().intersectAndStore("setValue","destSetValue","intersectValue");
    set = redisTemplate.opsForSet().members("intersectValue");
    System.out.println("通过intersectAndStore(K key, K otherKey, K destKey)方法将求出来的交集元素保存:" + set);

## 19.intersectAndStore(K key, Collection<K> otherKeys, K destKey)
	获取多个变量的交集并保存到最后一个参数上。

    redisTemplate.opsForSet().intersectAndStore("setValue",list,"intersectListValue");
    set = redisTemplate.opsForSet().members("intersectListValue");
    System.out.println("通过intersectAndStore(K key, Collection<K> otherKeys, K destKey)方法将求出来的交集元素保存:" + set);

## 20.union(K key, K otherKey)
	获取2个变量的合集。

    set = redisTemplate.opsForSet().union("setValue","destSetValue");
    System.out.println("通过union(K key, K otherKey)方法获取2个变量的合集元素:" + set);

## 21.union(K key, Collection<K> otherKeys)
	获取多个变量的合集。

    set = redisTemplate.opsForSet().union("setValue",list);
    System.out.println("通过union(K key, Collection<K> otherKeys)方法获取多个变量的合集元素:" + set);

## 22.unionAndStore(K key, K otherKey, K destKey)
	获取2个变量合集后保存到最后一个参数上。
    
    redisTemplate.opsForSet().unionAndStore("setValue","destSetValue","unionValue");
    set = redisTemplate.opsForSet().members("unionValue");
    System.out.println("通过unionAndStore(K key, K otherKey, K destKey)方法将求出来的交集元素保存:" + set);

## 23.unionAndStore(K key, Collection<K> otherKeys, K destKey)
	获取多个变量的合集并保存到最后一个参数上。

    redisTemplate.opsForSet().unionAndStore("setValue",list,"unionListValue");
    set = redisTemplate.opsForSet().members("unionListValue");
    System.out.println("通过unionAndStore(K key, Collection<K> otherKeys, K destKey)方法将求出来的交集元素保存:" + set);

---
# RedisTemplate常用集合-opsForZSet

## 1.add(K key, V value, double score)
	添加元素到变量中同时指定元素的分值。

    redisTemplate.opsForZSet().add("zSetValue","A",1);
    redisTemplate.opsForZSet().add("zSetValue","B",3);
    redisTemplate.opsForZSet().add("zSetValue","C",2);
    redisTemplate.opsForZSet().add("zSetValue","D",5);

## 2.range(K key, long start, long end)
	获取变量指定区间的元素。

    Set zSetValue = redisTemplate.opsForZSet().range("zSetValue",0,-1);
    System.out.println("通过range(K key, long start, long end)方法获取指定区间的元素:" + zSetValue);

## 3.rangeByLex(K key, RedisZSetCommands.Range range)
	 用于获取满足非score的排序取值。这个排序只有在有相同分数的情况下才能使用，如果有不同的分数则返回值不确定。

    RedisZSetCommands.Range range = new RedisZSetCommands.Range();
    //range.gt("A");
    range.lt("D");
    zSetValue = redisTemplate.opsForZSet().rangeByLex("zSetValue", range);
    System.out.println("通过rangeByLex(K key, RedisZSetCommands.Range range)方法获取满足非score的排序取值元素:" + zSetValue);

## 4.rangeByLex(K key, RedisZSetCommands.Range range, RedisZSetCommands.Limit limit)
	用于获取满足非score的设置下标开始的长度排序取值。

    RedisZSetCommands.Limit limit = new RedisZSetCommands.Limit();
    limit.count(2);
    //起始下标为0
    limit.offset(1);
    zSetValue = redisTemplate.opsForZSet().rangeByLex("zSetValue", range,limit);
    System.out.println("通过rangeByLex(K key, RedisZSetCommands.Range range, RedisZSetCommands.Limit limit)方法获取满足非score的排序取值元素:" + zSetValue);

## 5.add(K key, Set<ZSetOperations.TypedTuple<V>> tuples)
	通过TypedTuple方式新增数据。

    ZSetOperations.TypedTuple<Object> typedTuple1 = new DefaultTypedTuple<Object>("E",6.0);
    ZSetOperations.TypedTuple<Object> typedTuple2 = new DefaultTypedTuple<Object>("F",7.0);
    ZSetOperations.TypedTuple<Object> typedTuple3 = new DefaultTypedTuple<Object>("G",5.0);
    Set<ZSetOperations.TypedTuple<Object>> typedTupleSet = new HashSet<ZSetOperations.TypedTuple<Object>>();
    typedTupleSet.add(typedTuple1);
    typedTupleSet.add(typedTuple2);
    typedTupleSet.add(typedTuple3);
    redisTemplate.opsForZSet().add("typedTupleSet",typedTupleSet);
    zSetValue = redisTemplate.opsForZSet().range("typedTupleSet",0,-1);
    System.out.println("通过add(K key, Set<ZSetOperations.TypedTuple<V>> tuples)方法添加元素:" + zSetValue);

## 6.rangeByScore(K key, double min, double max)
	根据设置的score获取区间值。

    zSetValue = redisTemplate.opsForZSet().rangeByScore("zSetValue",1,2);
    System.out.println("通过rangeByScore(K key, double min, double max)方法根据设置的score获取区间值:" + zSetValue);

## 7.rangeByScore(K key, double min, double max,long offset, long count)
	根据设置的score获取区间值从给定下标和给定长度获取最终值。

    zSetValue = redisTemplate.opsForZSet().rangeByScore("zSetValue",1,5,1,3);
    System.out.println("通过rangeByScore(K key, double min, double max, long offset, long count)方法根据设置的score获取区间值:" + zSetValue);

## 8.rangeWithScores(K key, long start, long end)
	 获取RedisZSetCommands.Tuples的区间值。

    Set<ZSetOperations.TypedTuple<Object>> typedTupleSet = redisTemplate.opsForZSet().rangeWithScores("typedTupleSet",1,3);
    Iterator<ZSetOperations.TypedTuple<Object>> iterator = typedTupleSet.iterator();
    while (iterator.hasNext()){
	    ZSetOperations.TypedTuple<Object> typedTuple = iterator.next();
	    Object value = typedTuple.getValue();
	    double score = typedTuple.getScore();
	    System.out.println("通过rangeWithScores(K key, long start, long end)方法获取RedisZSetCommands.Tuples的区间值:" + value + "---->" + score );
    }

## 9.rangeByScoreWithScores(K key, double min, double max)
	获取RedisZSetCommands.Tuples的区间值通过分值。

    Set<ZSetOperations.TypedTuple<Object>> typedTupleSet = redisTemplate.opsForZSet().rangeByScoreWithScores("typedTupleSet",5,8);
    iterator = typedTupleSet.iterator();
    while (iterator.hasNext()){
	    ZSetOperations.TypedTuple<Object> typedTuple = iterator.next();
	    Object value = typedTuple.getValue();
	    double score = typedTuple.getScore();
	    System.out.println("通过rangeByScoreWithScores(K key, double min, double max)方法获取RedisZSetCommands.Tuples的区间值通过分值:" + value + "---->" + score );
    }

## 10.rangeByScoreWithScores(K key, double min, double max, long offset, long count)
	获取RedisZSetCommands.Tuples的区间值从给定下标和给定长度获取最终值通过分值。

    Set<ZSetOperations.TypedTuple<Object>> typedTupleSet = redisTemplate.opsForZSet().rangeByScoreWithScores("typedTupleSet",5,8,1,1);
    iterator = typedTupleSet.iterator();
    while (iterator.hasNext()){
	    ZSetOperations.TypedTuple<Object> typedTuple = iterator.next();
	    Object value = typedTuple.getValue();
	    double score = typedTuple.getScore();
	    System.out.println("通过rangeByScoreWithScores(K key, double min, double max, long offset, long count)方法获取RedisZSetCommands.Tuples的区间值从给定下标和给定长度获取最终值通过分值:" + value + "---->" + score );
    }

## 11.count(K key, double min, double max)
	获取区间值的个数。

    long count = redisTemplate.opsForZSet().count("zSetValue",1,5);
    System.out.println("通过count(K key, double min, double max)方法获取区间值的个数:" + count);

## 12.rank(K key, Object o)
	获取变量中元素的索引,下标开始位置为0。

    long index = redisTemplate.opsForZSet().rank("zSetValue","B");
    System.out.println("通过rank(K key, Object o)方法获取变量中元素的索引:" + index);

## 13.scan(K key, ScanOptions options)
	匹配获取键值对，ScanOptions.NONE为获取全部键值对；ScanOptions.scanOptions().match("C").build()匹配获取键位map1的键值对,不能模糊匹配。

    //Cursor<Object> cursor = redisTemplate.opsForSet().scan("setValue", ScanOptions.NONE);
    Cursor<ZSetOperations.TypedTuple<Object>> cursor = redisTemplate.opsForZSet().scan("zSetValue", ScanOptions.NONE);
	    while (cursor.hasNext()){
	    ZSetOperations.TypedTuple<Object> typedTuple = cursor.next();
	    System.out.println("通过scan(K key, ScanOptions options)方法获取匹配元素:" + typedTuple.getValue() + "--->" + typedTuple.getScore());
    }

## 14.score(K key, Object o)
	 获取元素的分值。

    double score = redisTemplate.opsForZSet().score("zSetValue","B");
    System.out.println("通过score(K key, Object o)方法获取元素的分值:" + score);

## 15.zCard(K key)
	获取变量中元素的个数。

    long zCard = redisTemplate.opsForZSet().zCard("zSetValue");
    System.out.println("通过zCard(K key)方法获取变量的长度:" + zCard);

## 16.incrementScore(K key, V value, double delta)
	修改变量中的元素的分值。

    double incrementScore = redisTemplate.opsForZSet().incrementScore("zSetValue","C",5);
    System.out.print("通过incrementScore(K key, V value, double delta)方法修改变量中的元素的分值:" + incrementScore);
    score = redisTemplate.opsForZSet().score("zSetValue","C");
    System.out.print(",修改后获取元素的分值:" + score);
    zSetValue = redisTemplate.opsForZSet().range("zSetValue",0,-1);
    System.out.println("，修改后排序的元素:" + zSetValue);

## 17.reverseRange(K key, long start, long end)
	索引倒序排列指定区间元素。

    zSetValue = redisTemplate.opsForZSet().reverseRange("zSetValue",0,-1);
    System.out.println("通过reverseRange(K key, long start, long end)方法倒序排列元素:" + zSetValue);

## 18.reverseRangeByScore(K key, double min, double max)
	倒序排列指定分值区间元素。

    zSetValue = redisTemplate.opsForZSet().reverseRangeByScore("zSetValue",1,5);
    System.out.println("通过reverseRangeByScore(K key, double min, double max)方法倒序排列指定分值区间元素:" + zSetValue);

## 19.reverseRangeByScore(K key, double min, double max, long offset, long count)
	倒序排列从给定下标和给定长度分值区间元素。

    zSetValue = redisTemplate.opsForZSet().reverseRangeByScore("zSetValue",1,5,1,2);
    System.out.println("通过reverseRangeByScore(K key, double min, double max, long offset, long count)方法倒序排列从给定下标和给定长度分值区间元素:" + zSetValue);

## 20.reverseRangeByScoreWithScores(K key, double min, double max)
	倒序排序获取RedisZSetCommands.Tuples的分值区间值。

    Set<ZSetOperations.TypedTuple<Object>> typedTupleSet = redisTemplate.opsForZSet().reverseRangeByScoreWithScores("zSetValue",1,5);
    iterator = typedTupleSet.iterator();
    while (iterator.hasNext()){
	    ZSetOperations.TypedTuple<Object> typedTuple = iterator.next();
	    Object value = typedTuple.getValue();
	    double score1 = typedTuple.getScore();
	    System.out.println("通过reverseRangeByScoreWithScores(K key, double min, double max)方法倒序排序获取RedisZSetCommands.Tuples的区间值:" + value + "---->" + score1 );
    }

## 21.reverseRangeByScoreWithScores(K key, double min, double max, long offset, long count)
	倒序排序获取RedisZSetCommands.Tuples的从给定下标和给定长度分值区间值。

    Set<ZSetOperations.TypedTuple<Object>> typedTupleSet = redisTemplate.opsForZSet().reverseRangeByScoreWithScores("zSetValue",1,5,1,2);
    iterator = typedTupleSet.iterator();
    while (iterator.hasNext()){
	    ZSetOperations.TypedTuple<Object> typedTuple = iterator.next();
	    Object value = typedTuple.getValue();
	    double score1 = typedTuple.getScore();
	    System.out.println("通过reverseRangeByScoreWithScores(K key, double min, double max, long offset, long count)方法倒序排序获取RedisZSetCommands.Tuples的从给定下标和给定长度区间值:" + value + "---->" + score1 );
    }

## 22.reverseRangeWithScores(K key, long start, long end)
	索引倒序排列区间值。

    Set<ZSetOperations.TypedTuple<Object>> typedTupleSet = redisTemplate.opsForZSet().reverseRangeWithScores("zSetValue",1,5);
    iterator = typedTupleSet.iterator();
	    while (iterator.hasNext()){
	    ZSetOperations.TypedTuple<Object> typedTuple = iterator.next();
	    Object value = typedTuple.getValue();
	    double score1 = typedTuple.getScore();
	    System.out.println("通过reverseRangeWithScores(K key, long start, long end)方法索引倒序排列区间值:" + value + "----->" + score1);
    }

## 23.reverseRank(K key, Object o)
	获取倒序排列的索引值。

    long reverseRank = redisTemplate.opsForZSet().reverseRank("zSetValue","B");
    System.out.println("通过reverseRank(K key, Object o)获取倒序排列的索引值:" + reverseRank);

## 24.intersectAndStore(K key, K otherKey, K destKey)
	获取2个变量的交集存放到第3个变量里面。

    redisTemplate.opsForZSet().intersectAndStore("zSetValue","typedTupleSet","intersectSet");
    zSetValue = redisTemplate.opsForZSet().range("intersectSet",0,-1);
    System.out.println("通过intersectAndStore(K key, K otherKey, K destKey)方法获取2个变量的交集存放到第3个变量里面:" + zSetValue);

## 25.intersectAndStore(K key, Collection<K> otherKeys, K destKey)
	获取多个变量的交集存放到第3个变量里面。

    List list = new ArrayList();
    list.add("typedTupleSet");
    redisTemplate.opsForZSet().intersectAndStore("zSetValue",list,"intersectListSet");
    zSetValue = redisTemplate.opsForZSet().range("intersectListSet",0,-1);
    System.out.println("通过intersectAndStore(K key, Collection<K> otherKeys, K destKey)方法获取多个变量的交集存放到第3个变量里面:" + zSetValue);

## 26.unionAndStore(K key, K otherKey, K destKey)
	获取2个变量的合集存放到第3个变量里面。

    redisTemplate.opsForZSet().unionAndStore("zSetValue","typedTupleSet","unionSet");
    zSetValue = redisTemplate.opsForZSet().range("unionSet",0,-1);
    System.out.println("通过unionAndStore(K key, K otherKey, K destKey)方法获取2个变量的交集存放到第3个变量里面:" + zSetValue);

## 27.unionAndStore(K key, Collection<K> otherKeys, K destKey)
	获取多个变量的合集存放到第3个变量里面。

    redisTemplate.opsForZSet().unionAndStore("zSetValue",list,"unionListSet");
    zSetValue = redisTemplate.opsForZSet().range("unionListSet",0,-1);
    System.out.println("通过unionAndStore(K key, Collection<K> otherKeys, K destKey)方法获取多个变量的交集存放到第3个变量里面:" + zSetValue);

## 28.remove(K key, Object... values)
	批量移除元素根据元素值。

    long removeCount = redisTemplate.opsForZSet().remove("unionListSet","A","B");
    zSetValue = redisTemplate.opsForZSet().range("unionListSet",0,-1);
    System.out.print("通过remove(K key, Object... values)方法移除元素的个数:" + removeCount);
    System.out.println(",移除后剩余的元素:" + zSetValue);

## 29.removeRangeByScore(K key, double min, double max)
	根据分值移除区间元素。

    removeCount = redisTemplate.opsForZSet().removeRangeByScore("unionListSet",3,5);
    zSetValue = redisTemplate.opsForZSet().range("unionListSet",0,-1);
    System.out.print("通过removeRangeByScore(K key, double min, double max)方法移除元素的个数:" + removeCount);
    System.out.println(",移除后剩余的元素:" + zSetValue);

## 30.removeRange(K key, long start, long end)
	根据索引值移除区间元素。

    removeCount = redisTemplate.opsForZSet().removeRange("unionListSet",3,5);
    zSetValue = redisTemplate.opsForZSet().range("unionListSet",0,-1);
    System.out.print("通过removeRange(K key, long start, long end)方法移除元素的个数:" + removeCount);
    System.out.println(",移除后剩余的元素:" + zSetValue);

# RedisTemplate实践一

参赛号的自增，利用自增获取参赛号的值，写入对象存库。
	
    //参赛号为空，且起始值大于-1
    if (StringUtils.isEmpty(enrollMatch.getSeq()) && match.getSeqStart() > -1) {
			//第一次参赛证生成
			//判断redis里面是否有对应的比赛的key,如果没有，
			//执行redisTemplate.opsForHash().increment("match_seq", match.get_id(), match.getSeqStart());
			//在起始参赛号自增1。
			//redisTemplate.opsForHash().hasKey("match_seq", match.get_id())
			if (!redisTemplate.opsForHash().hasKey("match_seq", match.get_id()))
				redisTemplate.opsForHash().increment("match_seq", match.get_id(), match.getSeqStart());
			//非第一次
			long num = redisTemplate.opsForHash().increment("match_seq", match.get_id(), 1);
			String numStr = "" + num;
			while (numStr.length() < 5) // 凑够长度5
				numStr = "0" + numStr;
			String seq = match.getSeqPrefix() + numStr;
	}

参赛证分配

数据结构：系统可以针对比赛设置赛区；赛区里设置考场，如第一考场，第二考场等；考场里设置有座位号，每个考场座位号从1开始，座位号个数可以系统设置。

需求：给每个参赛选手分配座位号。

实现过程：

    //查询所有需要分配考场座位的记录。根据比赛阶段（复赛，线下形式），已交报名费，已选赛点，未分配考场。
    List<MenrollPhase> menrollPhaseList = mongoTemplate.find(Query.query(Criteria.where("phaseId").is(phaseId).and("state").is(2).and("matchPlaceId").ne(null).ne("").and("matchPlaceRoomId").in(null,"")).with(new Sort("_id", "ASC")), MenrollPhase.class);
	//查询比赛是否设置赛区、需要分配考场的记录不为空且size大于0
	if(mphase.getMatchPlaceIds()!=null&&menrollPhaseList!=null&&menrollPhaseList.size()>0){
	lineNumber=menrollPhaseList.size();//要生成参赛证总数
		//查询该比赛所有赛区
		List<MmatchPlace> mmatchPlaceList = mongoTemplate.find(Query.query(Criteria.where("_id").in(mphase.getMatchPlaceIds())), MmatchPlace.class);
		if(mmatchPlaceList!=null&&mmatchPlaceList.size()>0){
			ListOperations<String, Object> lo = redisTemplate.opsForList();
			//循环遍历赛区，将考场设置到赛区中
			for(MmatchPlace mmatchPlace : mmatchPlaceList){
				List<MmatchPlaceRoom> mmatchPlaceRoomList = mongoTemplate.find(Query.query(Criteria.where("placeId").in(mmatchPlace.get_id())).with(new Sort("order", "ASC")), MmatchPlaceRoom.class);
				mmatchPlace.setMmatchPlaceRooms(mmatchPlaceRoomList);
			}
			//参赛证模板
			String[] temp = mphase.getCertTpl().split("\\|");
			//文件名
			String fileName = temp[2];
			
			String quchu=null;
			//加锁，分布式环境下只能有一个线程去考场获取座位号
			boolean groupAbsent = redisTemplate.opsForValue().setIfAbsent("placeRoom_" + mphase.get_id(), "roomsuo");
			//groupAbsent为true  可以执行当前代码
			if(groupAbsent){
				//遍历赛区
				for(MmatchPlace mmatchPlace : mmatchPlaceList){
					//比赛和赛区id共同组成key
					String mmatchPlaceIdKey = "matchPlace_" +mphase.get_id()+mmatchPlace.get_id();
					//首次分配
					if(!redisTemplate.hasKey(mmatchPlaceIdKey)&&mmatchPlace.getMmatchPlaceRooms()!=null&&mmatchPlace.getMmatchPlaceRooms().size()>0){
						//遍历赛区下的考场，考场id和座位号共同组成value，赛区为key,考场和座位为value中间以“，”隔开，存入
						//redis。
						for(MmatchPlaceRoom mmatchPlaceRoom : mmatchPlace.getMmatchPlaceRooms()){
							for(int i=0;i<mmatchPlaceRoom.getCounts();i++){
								String placeValue = mmatchPlaceRoom.get_id()+","+(i+1);
								lo.leftPush(mmatchPlaceIdKey, placeValue);//存入redis
							}
						}
					}
				}
				checkStatus=1;//准备考场已经结束
				//开始分考场
				for(MenrollPhase menrollPhase : menrollPhaseList){
					if(!StringUtils.isEmpty(menrollPhase.getMatchPlaceId())&&StringUtils.isEmpty(menrollPhase.getMatchPlaceRoomId())){
						for(MmatchPlace mmatchPlace : mmatchPlaceList){
							if(mmatchPlace.get_id().equals(menrollPhase.getMatchPlaceId())){
								//取出分配好的座位
								quchu = (String) lo.rightPop("matchPlace_" +mphase.get_id()+mmatchPlace.get_id());
								//redis里面没有说明已经取出了。
								if(StringUtils.isEmpty(quchu)){
									continue;
								}
								String[] tempNum = quchu.split(",");
								String roomId = tempNum[0];//考场Id
								String numBer = tempNum[1];//座位号
								for(MmatchPlaceRoom mmatchPlaceRoom : mmatchPlace.getMmatchPlaceRooms()){
									if(mmatchPlaceRoom.get_id().equals(roomId)){
										//将取出的考场和座位号存入对象写入数据库
										menrollPhase.setMatchPlaceRoomId(mmatchPlaceRoom.get_id());
										menrollPhase.setMatchPlaceRoomNum(Integer.valueOf(numBer));
										mongoTemplate.save(menrollPhase);
										menrollPhase.setMatchPlace(mmatchPlace);
										menrollPhase.setMatchPlaceRoom(mmatchPlaceRoom);
									}
								}
							}
						}
					}
				    }
				}
				//删除锁
				redisTemplate.delete("placeRoom_" + mphase.get_id());
				
				//开始生成参赛证
				try{
					caseCount=0;
					this.getDetails(menrollPhaseList);
					//方法一：使用Windows系统字体(TrueType)  
			        BaseFont baseFont = BaseFont.createFont(path+"WEB-INF/template/SIMSUN.TTC,1",BaseFont.IDENTITY_H,BaseFont.NOT_EMBEDDED); 
					for(MenrollPhase menrollPhase : menrollPhaseList){
						if(!StringUtils.isEmpty(menrollPhase.getMatchPlaceId())
								&&!StringUtils.isEmpty(menrollPhase.getMatchPlaceRoomId())
								&&menrollPhase.getMatchPlaceRoomNum()!=0){
							boolean isOk = checkedCarState(menrollPhase);
					        if(!isOk){
					        	continue;
					        }
							PdfReader reader = new PdfReader(fileName);
							ByteArrayOutputStream bos = new ByteArrayOutputStream();
							PdfStamper ps = new PdfStamper(reader, bos);
					        AcroFields fields = ps.getAcroFields();
					        fields.addSubstitutionFont(baseFont);
							Map<String, String> map = new HashMap<String, String>();
					        fillMap(map,menrollPhase);
					        fillData(fields, map);
							ps.setFormFlattening(true);
							ps.close();
							byte[] bytes = bos.toByteArray();
							String base = Base64Util.encode(bytes).trim();
							redisTemplate.opsForHash().put("cansai_"+phaseId,menrollPhase.get_id(), base);
							caseCount++;
						}
					}
					contectStatus=1;//参赛证生成结束
				}catch (IOException|DocumentException e) {
				 e.printStackTrace();
			 }
		   }
		}


# Redis分布式锁解决抢购问题

先新建一个RedisLock类：

	public class RedisService {

	    @Autowired
	    private RedisTemplate stringRedisTemplate;
	
	    /***
	     * 加锁
	     * @param key
	     * @param value 当前时间+超时时间
	     * @return 锁住返回true
	     */
	    public boolean lock(String key,String value){
	        if(stringRedisTemplate.opsForValue().setIfAbsent(key,value)){//setNX 返回boolean
	            return true;
	        }
	        //如果锁超时 ***
	        String currentValue = stringRedisTemplate.opsForValue().get(key);
	        if(!StringUtils.isEmpty(currentValue) && Long.parseLong(currentValue)<System.currentTimeMillis()){
	            //获取上一个锁的时间
	            String oldvalue  = stringRedisTemplate.opsForValue().getAndSet(key,value);
	            if(!StringUtils.isEmpty(oldvalue)&&oldvalue.equals(currentValue)){
	                return true;
	            }
	        }
	        return false;
	    }
	    /***
	     * 解锁
	     * @param key
	     * @param value
	     * @return
	     */
	    public void unlock(String key,String value){
	        try {
	            String currentValue = stringRedisTemplate.opsForValue().get(key);
	            if(!StringUtils.isEmpty(currentValue)&&currentValue.equals(value)){
	                stringRedisTemplate.opsForValue().getOperations().delete(key);
	            }
	        } catch (Exception e) {
	            log.error("解锁异常");
	        }
	    }
	}

首先，锁的value值是当前时间加上过期时间的时间戳，Long类型。首先看到用setiFAbsent方法也就是对应的SETNX，在没有线程获得锁的情况下可以直接拿到锁，并返回true也就是加锁，最后没有获得锁的线程会返回false。 

最重要的是中间对于锁超时的处理，如果没有这段代码，当秒杀方法发生异常的时候，后续的线程都无法得到锁，也就陷入了一个死锁的情况。我们可以假设CurrentValue为A，并且在执行过程中抛出了异常，这时进入了两个value为B的线程来争夺这个锁，也就是走到了注释*的地方。currentValue==A，这时某一个线程执行到了getAndSet(key,value)函数(某一时刻一定只有一个线程执行这个方法，其他要等待)。这时oldvalue也就是之前的value等于A，在方法执行过后，oldvalue会被设置为当前的value也就是B。这时继续执行，由于oldValue==currentValue所以该线程获取到锁。而另一个线程获取的oldvalue是B，而currentValue是A，所以他就获取不到锁啦。

业务代码：

    private static final int TIMEOUT= 10*1000;
	@Transactional
	public void orderProductMockDiffUser(String productId){
	   long time = System.currentTimeMillions()+TIMEOUT;
	   if(!redislock.lock(productId,String.valueOf(time)){
	    throw new SellException(101,"换个姿势再试试")
	    }
	    //1.查库存
	    int stockNum  = stock.get(productId);
	    if(stocknum == 0){
	        throw new SellException(ProductStatusEnum.STOCK_EMPTY);
	        //这里抛出的异常要是运行时异常，否则无法进行数据回滚，这也是spring中比较基础的   
	    }else{
	        //2.下单
	        orders.put(KeyUtil.genUniqueKey(),productId);//生成随机用户id模拟高并发
	        sotckNum = stockNum-1;
	        try{
	            Thread.sleep(100);
	        } catch (InterruptedExcption e){
	            e.printStackTrace();
	        }
	        stock.put(productId,stockNum);
	    }
	    redisLock.unlock(productId,String.valueOf(time));
	}
