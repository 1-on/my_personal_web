---
title: ArrayList对比LinkedList
abbrlink: 5
categories: 学习笔记
tags: java知识点
updated: 2023-2-10
date: 2023-02-10
---



### 底层原理

- ArrayList 底层是基于数组实现的，ArrayList 类是一个可以动态修改的数组，与普通数组的区别就是它是没有固定大小的限制，我们可以添加或删除元素。

- LinkedList 底层是基于链表实现的，是一种线性表，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的地址。

正因为底层数据结构的不同，他们适用的场景不同，**ArrayList 更适合随机查找**，**LinkedList 更适合删除和添加**，查询、添加、删除的时间复杂度不同。

那么LinkedList的删除和添加真的一定比ArrayList快吗？

<!-- more -->

### 测试代码

```java
public class LinkListArrayListTest {

    public static final int dataSize = 100000;

    public static void main(String[] args) {
        ArrayList<Integer> arrayList = new ArrayList<>();
        LinkedList<Integer> linkedList = new LinkedList<>();

        Random random = new Random();

        for (int i = 0; i < dataSize; i++) {
            int value = random.nextInt(dataSize);
            arrayList.add(value);
            linkedList.add(value);
        }

        StopWatch stopWatch = new StopWatch();
        // 测试数据插入性能
        stopWatch.start();
        for (int i = 0; i < dataSize; i++) {
            arrayList.add(random.nextInt(dataSize), random.nextInt(dataSize));
        }
        stopWatch.stop();
        long lastTaskTimeMillis = stopWatch.getLastTaskTimeMillis();
        System.out.println("ArrayList Insert time " + lastTaskTimeMillis + "ms");
        // 测试数据插入性能
        stopWatch.start();
        for (int i = 0; i < dataSize; i++) {
            linkedList.add(random.nextInt(dataSize), random.nextInt(dataSize));
        }
        stopWatch.stop();
        lastTaskTimeMillis = stopWatch.getLastTaskTimeMillis();
        System.out.println("LinkedList Insert time " + lastTaskTimeMillis + "ms");

    }

}
```

### 控制台输出

```java
ArrayList Insert time 964ms
LinkedList Insert time 38919ms
```

通过测试ArrayList和LinkedList插入等量数据的用时对比，可以看出LinkedList的用时时间是远大于ArrayList的用时时间。

### 原因分析

链表需要跳转n次才能抵达第n个节点，才能在这个节点插入。所以链表的随机插入复杂度一样是O(N)。而时间差距就出在缓存上。

数组在内存中使用的是连续内存，而链表在内存中是分散的，由于现代CPU的优化，依次访问连续内存的性能会比跳跃访问随机内存快出10倍以上(未来差距会越来越大)。

