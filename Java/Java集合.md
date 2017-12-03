## HashMap深入理解

>HashMap是Java中比较常用的集合类，相比较其他几个比较常用的集合类来说，HashMap的底层实现比较复杂，所以面试的时候问的也就比较多。它根据HashCode的值存储数据，大部分情况下都能直接定位到值，所以具有很快的访问速度，但是遍历速度却不确定。HashMap最多只允许一条记录的键为null，允许多条记录的值为null。

### HashMap的数据结构

HashMap的底层是使用数组+链表的结构实现的，当链表的结构超过8时，链表会自动转化为红黑树来存储节点。这样做的目的是为了对HashMap进行元素的操作优化，在链表的长度过长时，会导致查询等速度减低，转化为红黑树则会有效的提升速度

![](http://upload-images.jianshu.io/upload_images/7779232-1a3b5f1c556ce0c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/552)


### HashMap的工作原理

1.Node内部类

HashMap中有一个非常重要的字段，Node[] table,即哈希桶数组，它是一个Node的数组。看一下Node的源码,它是一个内部类。

```Java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node

        Node(int hash, K key, V value, Node<K,V> next) { ... }
        public final K getKey(){ ... }
        public final V getValue() { ... }
        public final String toString() { ... }
        public final int hashCode() { ... }
        public final V setValue(V newValue) { ... }
        public final boolean equals(Object o) { ... }
}
```
从内部类中可以很明显了解到Node的具体功能，其中hash是用来存储该元素的hash值，key和value分别代表着元素的键值。

2.扩容机制
![](https://github.com/weiguangshuai/note/blob/master/334906-106.jpg)
