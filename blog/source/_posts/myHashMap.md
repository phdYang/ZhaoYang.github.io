---
abbrlink: myHashMap
title: 徒手撸轮子系列之HashMap——我们撸一个HashMap怎么样
date: 2021-04-27 23:17:19
cover: https://i.loli.net/2021/04/27/NVpejmMIXl2cq8w.jpg
tags:
 - 徒手撸轮子系列
---

# 徒手撸轮子系列之HashMap——我们撸一个HashMap怎么样



## 背景

反思我们在项目中，或者生活中，遇到的最多的一个结构，应用最为频繁的一个结构，应该是什么？

数组？链表？树？图？

如果，让我来选择一个皇冠，我愿意将它戴在散列表的头上。

散列表，就是我们熟知的Map。我们想象一下的他的结构，键值对的结构，在现在的代码社会里几乎无所不在，万物皆对象，对象就是变量名与值的对应，不同的是，对象的Key我们一般事先定义好。亦或者，现在极其普遍的Json结构，其本质也是键值对的结构。

那么，我们为什么会需要这样一个结构呢？

众所周知，数组是程序世界的基石。我甚至都不愿意把皇冠给它，因为它乃基石。数组的一大特点是连续内存存储，所以他有一个优势，即按照下标，我们读取一个数值的时间复杂度是o(1)。

但是数组有一个前提，按照下标。

可不可以有一个不需要固定下标，也可以实现随机o(1)复杂度的访问的数据结构？

当然有，这就是我们的重点——键值对，即我们需要一个维护key以及该key所对应的value的一个键值对的数据结构。



## 自制HashMap

### 弄清需求，搭建框架

在我个人浅薄的知识面下，我的第一反应是，键值对作为一个对象，因为我要存储一堆键值对，那么，很自然的会写出如下的定义：

```java
private Node[] kvArr = new Node[size];  

class Node<K,V> {
    private K key;
    private V value;
}
```

但，

我么假如使用上述结构，那么，我们如何实现根据K获得V这一功能呢？

```java
for(Node node:kvArr){
    if(node.k.equals(key))){
        return node.v;
    }
}
```

这是个什么鬼的数据结构 

所以大家可以明白了，底层使用的数组，却得到了更差劲儿的性能？

那么我们可不可以优化增加的逻辑，使得Node不是随便加？



### 测试逻辑，完善框架

要明白的一点，要想使得查询不遍历，那么增加也不得随意，必须按照一定的规则

什么规则？

最简单的角度，我是不是可以把k与下标做一个函数关联？

比如，经过计算，```f(k) = i``` 的放在下标```i```的位置里

那么，我们可以增加一个如上函数：

```java
private int hash(K key) {
    return key.hashCode() % capacity;
}
```

这里需要解释两个东西

- hashCode(). hashCode是Object类的基本属性，用来对每一个对象计算一个数值
- capacity。我们最终得到的数值是需要放在长度为capacity的数组中的，所以需要取余，以免越界。至于capacity的取值，这里先xjb等于10

所以，无论是增加还是查询，都需要借助该hash函数，所以，灵魂是谁，一目了然

增加功能

```java
public void put(K key, V value) {
    Node<K,V> node = new Node<K, V>(key, value);
    kvArr[hash(key)] = node;
}
```

查询功能

```java
public V getValue(K key) {
    return kvArr[hash(key)].getValue();
}
```

是不是觉得很完美

不

现在，才刚刚入门

这个所谓的优化，有两个极其大的漏洞

1. size如果一旦确定。用尽了怎么办？

2. hash值，相等怎么办



### 深入优化，完成框架

所以，我们遇到了hash结构的第一个核心问题，那就是hash冲突

如何解决hash冲突呢？

很简单的方式，在add函数里增加判断，如果计算该hash值所在数组有值，则增加到下一个位置，如果还有值，继续。



```java
public void put(K key, V value) {
    Node<K,V> node = new Node<K, V>(key, value);
    int hash = hash(key);
    //循环判断是否有值
    while (kvArr[hash] != null) {
        hash = (hash + 1) % capacity;
    }
    kvArr[hash] = node;
}
```

!!!

当我们修改add函数的同时，不要忘记了，查询和新增永远是相辅相成的。

那么我们查询也需要依赖冲突算法，那么我们该如何解决？

这里有一个直接的想法，就是如果在add的时候，解决了几次冲突，我们可以记录下来冲突的次数就好了？

这里我脑子里出现的第一个思路是：记录下来次数，当我get的时候按照既定次数去计算即可。

但，次数记载哪里？

我们在add的时候，解决冲突的次数是不确定的，他是一个随机往后寻找位置的一个过程。而我们在查找的时候，只有一个key以及根据这个key第一次hash计算出来的Node的节点。

那么，记录次数似乎不可以，我们还有什么？

我们有key以及node节点的key啊！

key的值一样不就可以了？！！！

所以，代码就简单了：

```java
public V getValue(K key) {
    int hash = hash(key);
    while (!kvArr[hash].getKey().equals(key)) {
        hash = (hash + 1) % capacity;
    }
    return kvArr[hash].getValue();
}
```

聪明的小伙伴，我猜你一定知道了些什么。

是的，这就是大名鼎鼎的**散列表之线性探测法**



### 最终结果，落地可用

上述解决了我们的一个核心问题，那就是hash冲突，实际上这属于hash函数的设计范畴。

还有另外一个核心问题，我们的初始化size用尽了，怎么办？

扩容呀

在扩容的时候，我们在这里需要提前明确两个概念：

- size. size指的是当前Map中已经有KV对的数量
- capacity. capacity的含义是，目前Map可以承载的最大KV对的数量

明确了这两个概念之后，所谓扩容，指的是，当size==capacity的时候或者说逼近，扩大capacity的容量

那这里需要首先提供，size的计算方法，在每次执行put的时候，相当于size加了一。

所以，Map的增加函数：

```java
public void put(K key, V value) {
    Node<K,V> node = new Node<K, V>(key, value);
    int hash = hash(key);
    if (this.size == capacity) {
        resize();
    }
    while (kvArr[hash] != null) {
        hash = (hash + 1) % capacity;
    }
    kvArr[hash] = node;
    this.size++;
}
```

明白了什么时候调用resize函数之后，我们接下来要做的是，集中精力编写resize函数。我们先来搞清楚几个关键点：

- capacity需要扩大多少？
- 原来的数据如何放到扩容后的数组中？

针对第一个问题，我们就简单的扩容2倍

对于第二个问题，我相信很多人的直接就是纯碎的拷贝。但实际上，我们如果能重新计算Hash，重新改变位置，才是更合适的。这里的问题就是，更好的解决hash冲突问题。其实hash的关键问题，就是如何巧妙的额设计，避免hash冲突。因为，hash冲突之后，按照线性探测的方法，我们需要更多的循环，这会严重影响HashMap的性能

所以，我们设计的代码如下：

```java
public void resize() {
    Node<K,V>[] tmp = new Node[this.capacity];
    System.arraycopy(kvArr, 0, tmp, 0, tmp.length);
    this.capacity = this.capacity * 2;
    this.kvArr = new Node[this.capacity];
    this.size = 0;
    for (Node<K,V> node:tmp) {
        this.put(node.getKey(), node.getValue());
    }
}
```

其实，我的思路是，先把当前的数组**浅拷贝**一份，然后用扩容后的数组替换掉当前的数组，此时，该数据capacity为之前的2倍，**size为0**



这就是扩容。



## 性能测试

目前为止，原则上我们的自制HashMap已经全部完成。 简单总结一下几个核心：

- **hash函数**。**解决key与下标相关联的问题**。在这里，我们采用的是Object自身的hascode来和capacity进行取余来计算。hashcode可以基本保证一个独有的值，取余只是为了让hashcode可以放置在数组之中
- **Node[]对象数组**。**解决如何存储的问题**。在这里，我们底层采用的是对象**数组**，我们将KV封装为一个节点对象，这是为了KV的一体性
- **Hash冲突**。**解决当两个key的hash值一样如何解决的问题**。在这里，使用的是**线性探测法**
- **扩容。解决当capacity不够用的问题。**扩容的核心是需要定义清楚size与capacity的含义



**如下表1，是HashMap与MyHashMap的对标：**

|                               | put 10万次 | get 10万次 |
| :---------------------------- | ---------- | ---------- |
| HashMap                       | **39ms**   | **11ms**   |
| MyHashMap -（数组+ 线性探测） | **1068ms** | **113ms**  |

在这里，有一个小小的插曲，我在进行大容量测试的时候，出现了一个数组越界的异常，日志显示我有一个负数的下标。

？？？

经排查，hashcode是有可能为负数的，其根本原因是hashcode的算法要不停的相乘然后累加，如：

```java
// from JDK String.java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

这里的根本原因是因为，int的范围在-2^31 ~ 2^31 - 1 即[-2147483648, 2147483647]



截止到此，我们的整个的Map就开发完毕了。

**但，我们的put性能慢了26倍，get性能慢10倍**



## 自制HashMap的性能追逐

首先，可以进行改进的地点有哪些？

哈哈哈，当然是核心要点那里：hash函数、冲突方法、底层存储以及扩容结构



### **hash函数的探索**

先开始一些专业点的东西，什么叫做Hash函数

**定义**：hash function 又称散列函数、哈希函数，指的是，将任意长度的数据映射到有限长度的域上。映射后的值称为哈希值。

**性质**：

​        1. 如果两个哈希值不同，那么原始输入一定不同；但哈希值相同，原始输入不一定相同，这种情况称之为“哈希碰撞”。一个好的hash函数的设计很少出现哈希碰撞。

​        2. 哈希值就有不可逆性，没有办法用来逆向计算原来的数值。其根本原因在于性质1，同一个hash值对应的输入有很多。

**应用**：保护资料，如SHA-256；确保传递真实的信息；散列表；错误校正，如冗余校验；语音识别，如MD5；

这里，我们采用Java 8中的Hash函数来进行测试。

```java
private int hash(K key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这个依旧借助了hashCode来计算hash值，即 h^(h>>>16)

其中，>>>  表示的是无符号右移，即，忽略符号为，空位都已0补齐。

举例，如10的二进制为1010，那么10 >>> 2 指的是右移2位，即，0010 = 2 

之后，在使用的时候，需要进一步和当前capacity进行求 **与**

```java
int hash = (capacity - 1) & hash(key);
```

PS： 这里在测试get10万次的时候，才测试出来一个bug。当前的put的逻辑，先计算了hash值，再进行扩容，扩容之后整个hash函数已经打散了。但是，最终put到数组中的还是按照原来的Hash值去进行的。这样在进行get的时候，初始计算的Hash值不同，再进行进行线性探测的时候，轨迹完全不同，所以就会出现null的情况。

但令人痴迷的是，10万次仅有一次这种情况。



这次的比较结果，如表2所示：

|                                                | put 10万次  | get 10万次  |
| ---------------------------------------------- | ----------- | ----------- |
| HashMap                                        | **39ms**    | **11ms**    |
| MyHashMap -（数组+ 线性探测）                  | **1068ms**  | **113ms**   |
| MyHashMap - （数组 + 线性探测 + JDK8hash函数） | **25188ms** | **36445ms** |

怎么回事

我们明明改造了HashMap的hash函数，为什么性能更差呢？



这主要是因为我们的hash函数直接照抄jdk 却没有系统性的优化，比如，为什么hashCode要 右移16？我们这里的capacity=10，我们右移10是不是可以？

那么我们看看，这次的对比结果：

|                                                           | put 10万次  | get 10万次  |
| --------------------------------------------------------- | ----------- | ----------- |
| MyHashMap -（数组+ 线性探测+右移10位+capacity=10）        | **24936ms** | **24936ms** |
| MyHashMap - （数组 + 线性探测 + 右移16位 + capacaity=10） | **25188ms** | **36445ms** |

那，我们让capacity = 16呢？

神奇的事情发生了！！！！

|                                                              | put 10万次 | get 10万次 |
| ------------------------------------------------------------ | ---------- | ---------- |
| MyHashMap -（数组+ 线性探测+右移10位+capacity=10）           | 24936ms    | 24936ms    |
| MyHashMap - （数组 + 线性探测 + 右移16位 + capacaity=10）    | 25188ms    | 36445ms    |
| MyHashMap - （数组 + 线性探测 + 右移8位 + capacaity=8）      | **935ms**  | **116ms**  |
| MyHashMap - （数组 + 线性探测 + 右移16位 + capacaity=16）    | **710ms**  | **76ms**   |
| MyHashMap - （数组 + 线性探测 + 右移32位 + capacaity=32）    | 41983ms    | 68684ms    |
| MyHashMap -（数组+ 线性探测 + hashcode%capacity+ capacity=10） | 1068ms     | 113ms      |
| MyHashMap -（数组+ 线性探测 + hashcode%capacity+ capacity=16） | 995ms      | 98ms       |

从上述的对比，我们发现，适当的增加capacity能简单的提升性能，但单纯修改这个值，无济于事。

**当jdk的hash函数 配合 capacity = 16 可以极大的提升性能。**

这里的16，是一个非常有讲究的数据

好了，经过我们的借鉴，我们目前put性能慢了17倍，get性能慢了5倍



### hash冲突

我们当前使用的线性探测的方法，当冲突之后，效率及其低下。有没有其他更加优雅的冲突解决方法？

这里直接给一些解决方案的汇总，大致可以分为在散列表内解决的，称之为闭散列法，在散列表之外解决的，称之为开散列法，也称之为拉链法。

来了，我依旧借鉴 jdk 8的经典拉链法。

所谓拉链法，就是当hash冲突的时候，已当前的node节点为首位，不停往下扩展。所以，首先需要修改原始数据结构：

```JAVA
static class Node<K, V> {
    private final K key;
    private final V value;
    private Node<K,V> next;
}
```

其实，就是链表的结构

然后，当put和get的时候按照新的冲突方法:

```JAVA
    public void put(K key, V value) {
        int hash = (this.capacity - 1) & hash(key);
        Node<K,V> p = table[hash];
        if (p == null) {
            table[hash] = new Node<>(key, value, null);
        } else {
            while (p.next != null) {
                p = p.next;
            }
            p.next = new Node<>(key, value, null);
        }

        this.size++;
    }
    
    public V get(K key) {
        int hash = (this.capacity - 1) & hash(key);
        Node<K,V> p = table[hash];
        if (p == null) {
            return null;
        }
        if (p.getKey().equals(key)) {
            return p.getValue();
        }
        while (p.next != null) {
            p = p.next;
            if (p.getKey().equals(key)) {
                return p.getValue();
            }
        }
        return null;
    }
```

改造如此完美，而且，resize是不是不是必须的了？因为容量无穷无尽。当然了，经过接下来的测试，resize依旧需要，但他的作用变了。

第一波的测试：

|                                                              | put 10万次  | get 10万次  |
| ------------------------------------------------------------ | ----------- | ----------- |
| MyHashMap1-(数组+ 线性探测+hashCode%capacity+capacity=10)    | 989ms       | 106ms       |
| MyHashMap2-(数组 + 线性探测 + 右移16位 + capacaity=16)       | 681ms       | 66ms        |
| **MyHashMap3-(数组 + 拉链法(无resize)+ 右移16位 + capacaity=16)** | **14387ms** | **20365ms** |
| HashMap(JDK 8)                                               | 31ms        | 10ms        |

原因很简单，没有resize啊，太臃肿了，导致算完hash都需要循环遍历，这有违初衷o(1)啊

这里提出一个新的概念，我们如何衡量Hash函数的性能？这里需要首先计算平均查找长度（ASL），而ASL又依赖于装载因子，

**装载因子 = 填入表中的元素个数 / Hash表的长度**

性质：当装载因子越大，说明元素个数多，空闲位置少，冲突概率大，ASL就越大，性能下降。

所以，真正的扩容，应该在达到一定的装载因子时就要进行.

这里不再证明，选择JDK 8的装载因子 **0.75** 我们分别进行测试

|                                                              | put 10万次 | get 10万次 |
| ------------------------------------------------------------ | ---------- | ---------- |
| MyHashMap1-(数组+ 线性探测+hashCode%capacity+capacity=10+装载因子为1) | 1083ms     | 107ms      |
| MyHashMap1-(数组+ 线性探测+hashCode%capacity+capacity=10+装载因子为0.75) | **234ms**  | **135ms**  |

天啊，，，仅仅是修改一个扩容的条件而已，性能居然好到令人发指

ok, 我们接下来进行一个 完整的测试，同时针对拉链法进行扩容改造

|                                                              | put 10万次 | get 10万次 |
| ------------------------------------------------------------ | ---------- | ---------- |
| MyHashMap1-(数组+ 线性探测+hashCode%capacity+capacity=10+装载因子0.75) | 228ms      | 124ms      |
| MyHashMap2-(数组 + 线性探测 + 右移16位 + capacaity=16++装载因子0.75) | 124ms      | 15ms       |
| **MyHashMap3-(数组 + 拉链法(有resize)+ 右移16位 + capacaity=16+装载因子0.75)** | **46ms**   | **14ms**   |
| HashMap(JDK 8)                                               | 31ms       | 10ms       |

神奇的事情，终于发生了。

我们的性能追上来了

**我们的性能终于在10万量级追上了jdk1.8**



## 总结

我们对全文进行一个粗略的总结

- 存储结构。**底层是对象数组。**对象用于标记KV对，数组用于将hash值与下标进行关联。
- Hash函数。解决key与Hash值一一映射的关系。借助Object的HashCode进行计算，JDK 8采用 h^(h>>>16)
- 初始容量。为了与Hash函数一致，初始容量为16。
- 装载因子。扩容有一个合适的时机，过早扩容，会因扩容影响性能；过晚扩容，则会增大Hash函数的冲突的概率，影响性能
- Hash冲突。本文选择了拉链法与线性探测法，我们可以看到，在put的性能拉链法优于线性探测法，而get性能差距不大





附录

1. 内部类的作用以及为什么需要静态内部类？
2. 深拷贝与浅拷贝
3. hashCode与int的范围
4. jdk 1.8的初始容量与hash函数的设计
5. 装载因子为0.75的数学含义
6. JDK 引入红黑树