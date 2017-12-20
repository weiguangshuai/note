# Java Nio

## buffer

### 属性

- 容量（Capacity）：缓存区所能容纳的数据元素的最大数量
- 上界（limit）：缓存区第一个不能被读或者写的元素
- 位置（position）：下一个要被读或者写的元素的索引
- 标记（mark）：一个备忘位置

### buffer存取过程

1.新创建一个buffer，创建成功以后结构如下

![](https://github.com/weiguangshuai/note/blob/master/%E5%9B%BE%E5%BA%8A/buffer1_20171212122936.png)

2.使用put向缓存区中添加元素，添加元素以后的结构如下

![](https://github.com/weiguangshuai/note/blob/master/%E5%9B%BE%E5%BA%8A/buffer2_20171212123135.png)

3.缓存区可以直接使用get()方法遍历来获取元素，但是使用前要进行翻转，使用flip()方法，翻转后如下

![](https://github.com/weiguangshuai/note/blob/master/%E5%9B%BE%E5%BA%8A/buffer3_20171212123321.png)


### 两个buffer进行比较

两个缓存区被认为相等的条件：

1.两个对象类型相同

2.两个对象都剩余同样数量的元素。buffer的容量不需要相同，而且缓冲区剩余数据的索引也不必相同，但是每个缓存区中剩余元素的数目必须相同

3.在每个缓存区中被get()函数返回的剩余数据元素序列必须一致

相同：

![](https://github.com/weiguangshuai/note/blob/master/%E5%9B%BE%E5%BA%8A/buffer4_20171212123954.png)

不同：

![](https://github.com/weiguangshuai/note/blob/master/%E5%9B%BE%E5%BA%8A/buffer5_20171212124029.png)



### 视图缓冲器
>当一个管理其他缓冲器所包含的的数据元素的缓冲器被创建时，这个缓冲器被成为视图缓冲器。大多数的视图缓冲器都是ByteBuffer的视图。

1.duplicate()函数创建一个与原始缓存区相似的缓存区。两个缓存区共享数据元素，拥有同样的容量，但每个缓冲区拥有各自的位置、上界和标记属性。对一个缓冲区内的数据元素所做的改变会反映到另外一个缓冲区上。这一副本缓冲区具有与原始缓冲区同样的数据视图。如果原始的缓冲区为只读,或者为直接缓冲区,新的缓冲区将继承这些属性。

```Java
CharBuffer buffer = CharBuffer.allocate(8);
buffer.position(3).limit(6).mark().position(5);
CharBuffer dupeBuffer  = buffer.duplicate();
buffer.clear();
```

2.分割缓冲区与复制相似，但是slice()创建一个从原始缓冲区的当前位置开始的新缓冲区，并且其容量是原始缓冲区的剩余元素数量(limit-position)。


## Channel
>Channel用于字节缓冲区和位于通道另一侧的实体（通常是一个文件或套接字）之间的有效传输

### 打开通道

通道有两种，分别是文件(file)通道和套接字(socket)通道；socket通道有可以直接创建新socket通道的工厂方法。FileChannel对象只能通过打开一个RandomAccessFile、FileInputStream和FileOutPutStream对象上调用getChannel()方法来获取，不能直接创建一个FileChannel对象。

### 使用通道

通道可以是单向的，也可以是双向的。一个实现了ReadableByteChannel接口的channel类，一个实现了WriteableByteChannel接口的channel类，这两个类都是单向的，只有都实现了这两个接口的类才是双向的

对对对
