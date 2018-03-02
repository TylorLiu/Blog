---
title: HashMap源码解读
date: 18/2/2 23:37
tags: JDK
category: note
---

逐行解读HashMap

先看put方法

````java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    //计算hash
    static final int hash(Object key) {
        int h;
        //用key的hashCode值右移16位，与hashCode异或，原值不大于2的16次方的话，相当于按位取反
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            // table是map的内置数组，若为空就要初始化
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            //通过n-1&hash计算该hash值在tab中的下标hash%(n-1)，若该位置数组元素为空，直接放入
            tab[i] = newNode(hash, key, value, null);
        else {
            //若原位置不为空，则分3种情况
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                //p.key与key相同，则把p的引用赋给e
                e = p;
            else if (p instanceof TreeNode)
                //如果p是个树节点，遍历树，如果存在键为key的Node就返回，不存在就挂树上
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //把p视为链表，遍历之，大致同上
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            //链表长度>=8(可以改jvm参数)时，把链表重构成红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    //如果原值为空或 onlyIfAbsent=false 则替换成新值
                    e.value = value;
                 //后处理，LinkedHashMap用来把node移至末尾
                afterNodeAccess(e);
                //返回原值
                return oldValue;
            }
        }
        //如果是新插入节点
        ++modCount;
        //size超过阈值
        if (++size > threshold)
            resize();
        //节点插入后处理
        afterNodeInsertion(evict);
        return null;
    }
````

整个put的过程如上，我们再来看下其中调用的几个方法，先看resize():
````java
    //返回node数组
    final Node<K,V>[] resize() {
        获取原数组大小
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //原阈值
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {//已经是最大值，只能扩容阈值
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if (//翻倍(newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                     //初始容量->最大容量/2之间，将容量&阈值翻倍
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // 原容量<=0,阈值>0时，将阈值作为新容量 initial capacity was placed in threshold
            newCap = oldThr;
        else {               // 初始化 zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            //默认初始化，阈值=0.75*容量
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //根据新容量计算新阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //创建新的Node数组
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            //分三种情况，将旧数组中的数据按hash装到新数组中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
````
resize()的过程无疑非常耗费资源(时间&内存空间)，那么什么情况下会触发resize呢？
查看代码，可以发现resize有7处引用，分布在put、compute等方法中，但是条件只有两个，tab数组为空或有新元素插入后map.size()>threshold。
为了避免频繁map频繁扩容，在创建Map应该评估数据量，并设置合理的初始容量，一般为2的n次方。

java8 HashMap中还加了红黑树实现，以及链表与红黑树相互转化的阈值，有兴趣的同学可以看下TreeNode代码。
