---
title: JAVA基础--集合
date: 2019-07-14 14:37:31
description: 
tags: 
- Java
- Interview
- Collection
categories:
- Java
- Basis
top:
---

{% cq %}集合的分类;➕ArrayList和vector的区别;➕HashMap与HashTable的区别;➕HashMap是怎么解决哈希冲突的;➕list和map的区别;➕set里的元素是不能重复的,那么用什么方法区分重复元素;➕ArrayList和linkedlist的存储特性;➕ArrayList集合中插入一万条数据,如何提高效率{% endcq %}

<!-- more -->

# __集合的分类__

------

主要分为collection,map.

- **collection**
  - list
    - **ArrayLis**t
    - linkedlist
    - vector(已过时,了解)
  - set
    - hashset
      - linkedhashset
    - treeset
- **map**
  - **hashmap**
    - linkedhashmap
  - hashtable
    - properties
  - concurrenthashmap
- treemap

<br>

# __ArrayList和vector的区别__

------

__共同点__:这两个类都实现了list接口,他们都是有序的集合(存储有序),底层是数组,允许元素重复和为null  

__区别__:  

- 同步性
  - ArrayList是非同步的
  - vector是同步的
  - 需要同步时一般通过collections工具构建同步的ArrayList
- 扩容大小
  - vector增长原来的一倍,ArrayList增长原来的0.5倍

<br>



# __HashMap与HashTable的区别？__

------

- HashMap没有考虑同步，是线程不安全的；Hashtable使用了synchronized关键字，是线程安全的；
- HashMap允许K/V都为null；后者K/V都不允许为null；
- HashMap继承自AbstractMap类；而Hashtable继承自Dictionary类；

HashMap的put方法的具体流程？

<br>

源码为:

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    HashMap.Node<K,V>[] tab; HashMap.Node<K,V> p; int n, i;
    // 1.如果table为空或者长度为0，即没有元素，那么使用resize()方法扩容    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 2.计算插入存储的数组索引i，此处计算方法同 1.7 中的indexFor()方法    // 如果数组为空，即不存在Hash冲突，则直接插入数组    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 3.插入时，如果发生Hash冲突，则依次往下判断    else {
        HashMap.Node<K,V> e; K k;
        // a.判断table[i]的元素的key是否与需要插入的key一样，若相同则直接用新的value覆盖掉旧的value        // 判断原则equals() - 所以需要当key的对象重写该方法        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // b.继续判断：需要插入的数据结构是红黑树还是链表        // 如果是红黑树，则直接在树中插入 or 更新键值对        else if (p instanceof HashMap.TreeNode)
            e = ((HashMap.TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 如果是链表，则在链表中插入 or 更新键值对        else {
            // i .遍历table[i]，判断key是否已存在：采用equals对比当前遍历结点的key与需要插入数据的key            //    如果存在相同的，则直接覆盖            // ii.遍历完毕后任务发现上述情况，则直接在链表尾部插入数据            //    插入完成后判断链表长度是否 > 8：若是，则把链表转换成红黑树            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 对于i 情况的后续操作：发现key已存在，直接用新value覆盖旧value&返回旧value        if (e != null) { // existing mapping for key            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 插入成功后，判断实际存在的键值对数量size > 最大容量    // 如果大于则进行扩容    if (++size > threshold)
        resize();
    // 插入成功时会调用的方法（默认实现为空）    afterNodeInsertion(evict);
    return null;
}
```

图示为:


![1561125478643.png](https://i.loli.net/2019/07/14/5d2adb9b887a564110.png)




<br>

# __HashMap是怎么解决哈希冲突的__

------

**什么是哈希？**
Hash，一般翻译为“散列”，也有直接音译为“哈希”的，这就是把任意长度的输入通过散列算法，变换成固定长度的输出，该输出就是散列值（哈希值）；这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，所以不可能从散列值来唯一的确定输入值。简单的说就是一种将任意长度的消息压缩到某一固定长度的消息摘要的函数。

所有散列函数都有如下一个基本特性：根据同一散列函数计算出的散列值如果不同，那么输入值肯定也不同。但是，根据同一散列函数计算出的散列值如果相同，输入值不一定相同。

__解决办法__

1. 使用链地址法（使用散列表）来链接拥有相同hash值的数据； 
2. 使用2次扰动函数（hash函数）来降低哈希冲突的概率，使得数据分布更平均；
3. 引入红黑树进一步降低遍历的时间复杂度，使得遍历更快；

<br>

# __list和map的区别__

------

- 存储结构不同
  - list是存储单列的集合
  - map是以键值对的形式存储数据的
- 元素是否重复
  - list允许元素重复
  - map不允许元素重复
- 是否有序
  - list集合元素是有序的
  - map集合元素是无序的

<br>

# __set里的元素是不能重复的,那么用什么方法来区分重复的元素,是==还是equals__

------

set集合实际上大都使用的是 __map集合的put方法来添加元素的__
以hashset元素为例,hashset里的元素不能重复
源码为
	    

```java
// 1. 如果key 相等  
if (p.hash == hash &&
    ((k = p.key) == key || (key != null && key.equals(k))))
    e = p;
// 2. 修改对应的value
   if (e != null) { // existing mapping for key
        V oldValue = e.value;
        if (!onlyIfAbsent || oldValue == null)
            e.value = value;
        afterNodeAccess(e);
        return oldValue;
   }
```

添加元素的时候,如果key(也就是对应的是set集合的元素)相等,那么修改value值.而在set集合中,value值仅仅是一个object对象罢了(该对象对set集合来说是无用的)

**Set集合如果添加的元素相同时，是根本没有插入的(仅修改了一个无用的value值)！从源码(HashMap)中也看出来，==和equals()方法都有使用！**

<br>

# __arraylist和linkedlist的存储特性__

------

ArrayList的底层是数组，LinkedList的底层是双向链表。

- ArrayList它支持以角标位置进行索引出对应的元素(随机访问)，而LinkedList则需要遍历整个链表来获取对应的元素。因此一般来说ArrayList的访问速度是要比LinkedList要快的
- ArrayList由于是数组，对于删除和修改而言消耗是比较大(复制和移动数组实现)，LinkedList是双向链表删除和修改只需要修改对应的指针即可，消耗是很小的。因此一般来说LinkedList的增删速度是要比ArrayList要快的

<br>

> ArrayList的增删未必就是比LinkedList要慢。
>
> - 如果增删都是在末尾来操作【每次调用的都是remove()和add()】，此时ArrayList就不需要移动和复制数组来进行操作了。如果数据量有百万级的时，速度是会比LinkedList要快的。
> - 如果删除操作的位置是在中间。由于LinkedList的消耗主要是在遍历上，ArrayList的消耗主要是在移动和复制上(底层调用的是arraycopy()方法，是native方法)。
>   - LinkedList的遍历速度是要慢于ArrayList的复制移动速度的
>   - 如果数据量有百万级的时，还是ArrayList要快。

<br>

# __ArrayList集合中插入一万条数据,如何提高效率__

------

ArrayList的默认初始容量为10，要插入大量数据的时候需要不断扩容，而扩容是非常影响性能的。因此，现在明确了10万条数据了，我们可以直接在初始化的时候就设置ArrayList的容量.

