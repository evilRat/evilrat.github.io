---
title: HashMap&ConcurrentHashMap
excerpt: ''
tags: [jdk]
categories: [jdk]
comments: true
date: 2019-08-18 00:30:52
---

## jdk1.7

### HashMap

1. HashMap默认大小是16，1 << 4
2. 可以使用构造方法`HashMap(int initialCapacity, float loadFactor)`来创建一个HashMap，实际上使用无参的构造函数也会调用这个有参的构造方法。这里只会将初始容量和加载因子赋值给对象的成员变量。这里并没有创建HashMap的中的Entry数组
3. 加载因子决定了什么时候扩充HashMap的容量。比如默认的加载因子是0.75L，也就是当容量已经占用到4/3的时候就会触发扩容。
4. 当我们调用`public V put(K key, V value)`的时候，会判断table（Entry数组）是否为空。如果为空，会调用`inflateTable(threshold)`，这里的threshold就是一开始创建HashMap的初始容量。在这里threshold会被处理成比它自身大的最小的2的幂，比如传3，这里就会是4，10这里就会是16。然后接下来就会创建一个Entry数组了。在inflateTable方法初始化了Entry数组后，就会判断key是否为空，如果为空就调用`putForNullKey(value)`，也就是说HashMap允许key为null
5. 为什么创建对象的时候容量一定要是2的幂呢？这和HashMap计算每个key的位置有关系。怎么确定应该放在什么位置？

```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

如果我们直接拿到key的hashcode，然后和数组长度进行取余，每次都会返回0~entry.length()之间的数字，也就可以唯一确认一个位置。

但是考虑到效率问题，jdk里并没有这么做，因为计算机进行二进制的位运算是最快的，所以这里选择了位与运算。

```java
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);
}
```

这里返回的是最终存在数组中的位置。这里是hash方法结果与长度-1做与运算。

先搞懂为什么是长度-1？
之前我们了解到数组长度肯定是2的幂，
比如key的hashcode的二进制为1010 1010
数组大小为8，也就是1000，减去1后就是0111
做与运算：
1010 1010
0000 0111
&
0000 0010

可以看出，数组长度为2的幂，减一后，低位一定全是1，也就是会把key的hashcode的低位保留下来，而且低几位也是确定的，而这低几位，不论key的hashcode是多少，肯定也是在0-entry.length-1之间。达到了和取余一样的效果，而且性能更好。

但是我们看到hash方法后面一顿位移操作，这又是为什么呢？

我们刚才所说的方法，如果直接那key的hashcode来做的话，那么大的数，只有最后几位有用，高位的值对最终的位置不会有影响，这样会有很大机率发生哈希冲突。

前面的hash方法中，在返回之前，做了右移和异或操作，这样的话把高位移到右边，再加上异或操作，使得高位也可以影响到最后产生的hash值，这样就可以减小发生hash冲突的可能性。这个操作也是因为`hash(Object k)`的参数可以是自定义的，那么这个自定义类的hashCode方法也是可以重写的，如果程序员设计的hashCode方法散列性不高，jdk源码中的右移和异或操作也可以增大散列性。
6. put方法中确定了位置之后，就开始遍历该位置的entry链表，如果有重复的，会将旧的值返回出去，并且把该位置的entry的value修改为新的value。如果没有重复的值，就回modCount++，然后调用`addEntry(hash, key, value, i)`，这个方法的逻辑就是，先判断size是否到达了阈值，也就是当前的容量大小*扩容因子，并且table[i]!=null，就会触发扩容（也就是table当前位置不为空的时候才会进行扩容，jdk1.8已经去掉），扩容是new一个新的entry数组，长度为之前容量的两倍，然后调用transfer方法，二重循环将旧数组中每个entry放到新的数组中，最后新数组会被赋值给旧数组变量，旧数组对象就会被回收。扩容处理完以后就会拿到数组中i位置的entry对象，然后调用new一个entry，next指向之前table[i]的entry，也就是放在了i位置的头部，然后把这个entry放在了table的i位置，之后size++，记录size是为了当调用map.size()的时候，更快的返回结果。
7. 到这里HashMap线程不安全的原因就明确了，当多线程中数组需要扩容时，new新数组的时候，就会发生一些预期外的结果。HashMap通过设置足够的容量可以避免线程不安全问题。

### HashTable

HashTable为了解决线程安全问题，在HashMap的基础上加入了锁，在方法中加入synchronized。但是这样做回严重影响效率，所以很少使用。

### ConcurrentHashMap

像HashTable，每个线程要去竞争同一把锁，这样回严重影响效率，但是如果分段加锁，可以减少竞争同一把锁的线程数，就可以有效的缓解线程之间的竞争，提高效率。

1. 增加了属性，并发级别，就是控制分多少段，默认16。
2. 增加了一个对象，segment，每个segment就像之前的一个HashMap。
ConcurrentHashMap中有个多个segment(小HashMap)，而且segment继承了ReentrantLock，也就是它自带锁，segment在执行put方法时会先去获取锁，操作完成后会释放锁。segment中有Entry数组。
3. ConcurrentHashMap在执行put方法时，要先计算出应该放在哪个segment中，然后调用这个segment的put方法。segment的put方法逻辑就像之前HashMap的put方法了。
4. 初始化segment数组的时候，要根据构造方法传入的容量和并发级别计算出每个segment多大。

```java
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
```

这里的ssize就是初始话segment数组的大小。计算ssize，就是得到最小的大于并发级别的2的幂。

- 容量为16，并发级别是8，那么segment数组的容量就是8
- 容量是16，并发级别是4，那么segment数组的容量就是4
- 容量为16，并发级别是5，那么segment数组的容量就是8

5. put时，会根据key计算出hash值，计算方法和HashMap稍有不同，不过思想是一样的。根据hash值得到要放在哪一个segment中，如果这个位置的segment是null，就要new一个segment。而一个segment有是一个entry数组，就要计算这个entry数组的大小应该是多少。不过这里很简单，就是整个ConcurrentHashMap的容量除以segment的容量

## jdk1.8

如果Map中的链表随着插入的entry的增多越来越长，就会导致每次操作越来越慢，因为链表是需要遍历的，每次put/get都要遍历判断链表上的每个entry。所以在jdk1.8引入了红黑树。

### HashMap

1. 当链表长度大于8时就会把这个链表修改为空黑树，再把新的entry加入到红黑树中。
2. 已经引入了红黑树，所以jdk1.8在计算hash的时候也是1.7的思路，但是没有那么多的右移了。
3. 1.8把之前的Entry改成了Node
4. 思路大致和1.7一致，putVal方法中，会根据key计算hash，然后计算出应该存放在table哪个位置，如果这个位置是null，就会new一个Node，放在table的该位置。如果这个位置不是null，先判断table数组中该位置的node是否key是一致的，如果是一致的，就直接替换这个node，如果不一致，就会先判断这个node是不是一个TreeNode(红黑树)，如果不是红黑树，也就是一个链表，有一个和1.7不一样的地方是，1.8会直接将这个node插入到链表的尾部，而1.7是插入到头部，插入到尾部是因为加入了红黑树，插入的时候一定要判断链表长度，好决定要不要转换成红黑树，所以遍历完以后不如直接插入到尾部，而且这样还有一个好处，就是扩容的时候，移动node时不会导致顺序变化，多线程的时候就不会导致循环链表的问题（因为每次扩容都变化node顺序，另外一个线程获取到的next可能时这个线程的上一个），插入完成后，会判断长度是否到达了8，如果是就会转换成红黑树。如果是红黑树，就插入到红黑树。

### ConcurrentHashMap

1. 1.8的ConcurrentHashMap基于1.8的HashMap来的。
2. 1.7的ConcurrentHashMap是分段加锁，即每个segment一把锁，而基于1.8HashMap的思想，每次操作一个链表或者红黑树，都会操作table上的node，也就是链表的头部，或者树的根节点，所以只在table的node上加锁就可以了。
3. 1.8的ConcurrentHashMap中就没有segment这个概念了，因为旨在table上的node加锁嘛。
4. 怎么加的锁呢？在操作ConcurrentHashMap时，获取到当前key在table上所在的位置的时候，就给table上这个位置的node对象加锁，使用synchronized给node对象加锁。这样减少了对象的创建，节省了内存。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```
