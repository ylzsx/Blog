## hashMap储存原理

jdk1.8以前是数组加单链表，jdk1.8以后，若单链表长度超过`TREEIFY_THRESHOLD = 8 `,则使用红黑树储存。

HashMap中存在一个内部类，代表每个节点。该内部类中，有四个属性，我们存储的是key和value。其中hash为通过key计算的hashcode值，next是指向下一个节点。

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
    // do soemthig
}
```

## 散列算法

把任意长度的输入，通过散列算法，变换成固定长度的输出，该输出为散列值。散列算法为了保证分布均匀。

HashMap中散列算法的实现是通过`hash()`函数

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```