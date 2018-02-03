---
title: HashMap源码解读
date: 18/2/2 23:37
tags: JDK HashMap
category: 源码
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
                //该key已经在map里了，先取到原node的引用，后面再处理
                e = p;
            else if (p instanceof TreeNode)
                //原位置是个树节点，把新Node丢上去
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //原位置是个链表，遍历之，遇到重复key就赋给e，遇不到就在末尾加上新节点
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            //链表长度大于7(可以改jvm参数)，树
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
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
````