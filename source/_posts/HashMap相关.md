---
title: HashMap相关
date: 2021-10-18 21:16:00
tags: 面基
category: 面基
---

## HashMap底层数据结构详解

JDK1.7及之前：数组+链表

JDK1.8:数组+链表+红黑树

HashMap添加元素时，会根据哈希值和数组长度来计算该元素put的位置。put元素会判断当前位置是否已经有元素，这样在多线程环境下，会发生数据覆盖，导致数据丢失。通常为了使元素分布均匀会使用取模运算，计算出index后，就会将改元素添加进去，但是会出现两个key的哈希值相同，这时候会产生哈希冲突，当哈希冲突时会产生链表的数据结构，冲突的元素会在该索引出以链表的形式保存。

但是当链表的长度过长时，其固有弊端就显示出来了，即查询效率低。

此时就引入了第三种数据结构，红黑树，红黑树是一刻接近于平衡的二叉树，远远比链表的查询料率高。但是如果链表的长度不到一定阈值，直接使用红黑树也不行，因为红黑树的自身维护代价也比较高，每插入一个元素都可能打破红黑树的平衡，这时候就需要对红黑树再平衡（左旋，右旋，重新着色）。

HashMap中数组的初始长度是16，默认的加载因子是0.75

数组一单达到容量的阈值就需要对数组进行扩容。扩容就意味着要进行数组的移动，数组一旦移动，每移动一次就要重新计算索引，这个过程牵扯大量元素的迁移，会大大影响效率。如果直接使用与运算，这个效率远远高于取模运算。


为什么链表长度大于等于8时转成了红黑树

遵循概率论里的泊松分布。链表中元素个数为8的概率已经非常小，红黑树平均查找长度是log（n），当长度为8时，平均查找长度为3，如果继续使用链表，平均查找长度为8、2=4，这时候才有转为树的必要。

## HashMap的遍历方式

迭代器遍历（iterator）

For Each方式遍历

Lambda表达式遍历

Streams Api遍历

迭代器entrySet

    @Test
    public void test001(){
        Map<String, Integer> map = new HashMap<>();
        map.put("1",1);
        map.put("2",2);
        Iterator<Map.Entry<String,Integer>> iterator = map.entrySet().iterator();
        while (iterator.hasNext()){
            Map.Entry<String,Integer> entry = iterator.next();
            System.out.println("key:"+entry.getKey()+",value:"+entry.getValue());
        }
    }

	当遍历时涉及到删除操作，建议使用iterator的remove方法。使用forEach会报错。

迭代器keySet

    @Test
    public void test002(){
        Map<String, Integer> map = new HashMap<>();
        map.put("11",1);
        map.put("22",2);
        Iterator<String> iterator = map.keySet().iterator();
        while (iterator.hasNext()){
            String key = iterator.next();
            System.out.println("key:"+key+",value:"+map.get(key));
        }
    }

ForEach EntrySet

    @Test
    public void test003(){
        Map<String, Integer> map = new HashMap<>();
        map.put("111",1);
        map.put("222",2);
        for (Map.Entry<String,Integer> entry:map.entrySet()
             ) {
            System.out.println("key:"+entry.getKey()+",value:"+entry.getValue());
        }
    }

	通过Map.entrySet遍历key和value，代码简洁高效，推荐使用


ForEach keySet(如果只需要获取所有的key，推荐使用，比entrySet遍历要快，代码简洁)

    @Test
    public void test004(){
        Map<String,Integer> map = new HashMap<>();
        map.put("1111",1);
        map.put("2222",2);
        for (String str:
             map.keySet()) {
            System.out.println("key:"+str+",value:"+map.get(str));
        }
    }
	
	根据键取值是耗时操作，不推荐使用


	如果只需要获取所有的value，推荐使用，比entrySet快，简洁

    @Test
    public void test004(){
        Map<String,Integer> map = new HashMap<>();
        map.put("1111",1);
        map.put("2222",2);
        for (String str:
             map.values()) {
            System.out.println("value:"+str);
        }
    }

Lambda表达式

    @Test
    public void test005(){
        Map<String,Integer> map = new HashMap<>();
        map.put("11111",1);
        map.put("22222",2);
        map.forEach((key,value)->{
            System.out.println("key:"+key+",value:"+value);
        });
    }

Streams API单线程

     @Test
    public void test006(){
        Map<String,Integer> map = new HashMap<>();
        map.put("111111",1);
        map.put("222222",2);
        map.entrySet().stream().forEach((entry)->{
            System.out.println("key:"+entry.getKey()+",value:"+entry.getValue());
        });
    }

Streams API多线程

    @Test
    public void test007(){
        Map<String,Integer> map = new HashMap<>();
        map.put("1111111",1);
        map.put("2222222",2);
        map.entrySet().parallelStream().forEach((entry)->{
            System.out.println("key:"+entry.getKey()+",value:"+entry.getValue());
        });
    }