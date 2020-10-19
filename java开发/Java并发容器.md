### HashMap

​	在JDK1.8之前，HashMap采用数组+链表实现，即使用链表处理冲突，同一hash值的节点都存储在一个链表里。但是当位于一个桶中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。而JDK1.8中，HashMap采用数组+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树，这样大大减少了查找时间。多线程环境下使用HashMap的put操作将可能导致Entry链表形成环形链表，导致死循环，应该使用ConcurrentHashMap

```Java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;
    // 默认的初始容量为16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶上的节点数大于8时将链表结构转化为红黑树结构
    static final int TREEIFY_THRESHOLD = 8;
    // 当桶上的节点数小于6时将树结构转化为链表结构
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶可以被转化为树结构的table的最小容量，否则，如果一个桶中有太多节点，就会重新调整table的大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<K,V>[] table;
    // 存放具体元素的集合
    transient Set<Map.Entry<K,V>> entrySet;
    // 存放元素的个数
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 填充因子
    final float loadFactor;
    
    // 返回大于cap的最小的二次幂值
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
    
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 如果table为空，执行resize进行扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 根据hash值得到插入的桶的索引，如果该桶为空，直接新建节点添加
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        // 如果桶不为空
        else {
            Node<K,V> e; K k;
            // 如果桶的头节点的key和要插入的key值相同，访问该节点
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 否则如果桶头节点是树结构，使用树的putTreeVal方法插入键值
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 如果桶节点是链表结构，遍历链表，找到key相同的节点，如果遍历到链表最后，那么在链表末尾添加新节点，如果桶的节点数大于阈值，则将链表结构转换为树结构或扩容
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 如果桶中存在key值相同的节点，返回旧值，并根据onlyIfAbsent参数选择是否更换新值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 如果table中实际元素个数大于阈值，扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
    
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 如果table已经初始化并且根据hash寻找table中的项也不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            // 桶中与第一个节点相等，返回这个节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            // 桶中不止一个节点
            if ((e = first.next) != null) {
                // 检查桶是否为树结构，如果是调用树的getTreeNode方法，返回节点
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 桶是链表建构，遍历链表节点寻找key相等的节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        // 没找到对应节点
        return null;
    }
    
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        // table已经初始化过
        if (oldCap > 0) {
           	// table的容量已经大于最大容量，将阈值设为最大整数，返回table
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 否则将新容量设为旧的2倍，如果新容量小于最大容量并且就容量大于默认初始容量，则
            // 双倍拓展新阈值
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        // 如果旧容量小于等于0，旧阈值大于0，将新容量置为旧阈值
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        // 否则新容量设为默认容量，新阈值设为负载因子乘以默认初始容量
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 如果新阈值为0，按照上面类似的规则设置新阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 更新阈值，创建新的table
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 开始进行扩容操作，将Node对象复制到新的hash桶数组中
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    // 将旧节点的引用设为空，方便gc
                    oldTab[j] = null;
                    // 如果e后面没有节点，直接在新table中根据hash值确定位置并将e指向该位置
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 如果e是个树结构，那么将调用e的split方法将树分拆成两个树放在新table的合适位置
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    // 链表结构
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            // 获取e的下一个节点
                            next = e.next;
                            // 判断下标有没有改变，采用尾插法
                            // e.hash & oldCap 可以看作e.hash%(oldCap+1),该步骤是为了计算位置是否需要移动。hashMap在扩容的时候，不需要像1.7那样重新计算hash值，只需要判断(2*oldCap-1)的最高位对应的hash位是0还是1就行了。是0下标不变，是1索引变成原下标+oldCap。这也是Java1.8优化的地方之一
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
                        // 原下标对应的链表
                        if (loTail != null) {
                            // 尾部节点next设置为nul
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 新下标对应的链表
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
}
```

​	采用上面的hash函数进行散列化的原因时：使用容量大小和key值进行hash的算法在开始的时候只会对地位进行计算，虽然容量的2进制高位一开始都是0，但是key的2进制高位通常是有值的，因此现在hash方法中将key的hashCode右移16位再与自身异或，使得高位也可参与hash，更大程度上减少了碰撞率。如图：

![](F:\mycode\knowledgeArrangement\java开发\hashCode.png)

HashMap在1.7和1.8中有哪些区别：

1. 1.7时链表结构使用头插法，1.8后使用尾插法。因为头插法时容易出现逆序且链表死循环问题。1.8后引入红黑树+尾插法，能够避免这个问题。
2. 扩容后数据存储位置的计算方式不一样。1.7直接用hash和2*oldCap-1进行&运算；1.8时只需要判断Hash值的新参与运算的位是0还是1，是0选择原始位置，是1则是原始位置+oldCap
3. JDK1.7的时候使用的是数组+ 单链表的数据结构。但是在JDK1.8及之后时，使用的是数组+链表+红黑树的数据结构（当链表的深度达到8的时候，也就是默认阈值，就会自动扩容把链表转成红黑树的数据结构来把时间复杂度从O（n）变成O（logN）提高了效率）

哈希表如何解决Hash冲突？

![](F:\mycode\knowledgeArrangement\java开发\hashnote1.png)

为什么HashMap具备下述特点：键-值（key-value）都允许为空、线程不安全、不保证有序、存储位置随时间变化？

![](F:\mycode\knowledgeArrangement\java开发\hashnote2.png)

为什么HashMap中String、Integer这样的包装类适合作为key键？

![](F:\mycode\knowledgeArrangement\java开发\hashnote3.png)

HashMap中的key若为Object类型，则需要实现哪些方法？

![](F:\mycode\knowledgeArrangement\java开发\hashnote4.png)

### ConcurrentHashMap

1.7和1.8的区别？

1. 1.7中ConcurrentHashMap采用锁分段技术，将数据分成一段一段地储存，然后给每一段数据配一把锁（segment），当一个线程占用锁访问其中一个数据段的时候，其他段的数据也能被其他线程访问。1.8中采用CAS+synchronized来代替Segment，（这样锁的粒度变小了，不是每次都要加锁，CAS尝试失败在加锁）
2. put方法。在1.7中，需要定位2次，segments[i]和segment中的table[i]。没获取到segment锁的线程，没权利进行put操作，但不会像HashTable那样挂起等待，而是去做一下put操作前的准备：定位table[i]的位置（哪个bin），通过首节点遍历链表找有没有相同的key，在执行前两部期间还不断自旋获取锁，超过64次线程挂起。在1.8中，根据rehash值定位，拿到table[i]的首节点first，如果为null，通过CAS操作将value写入；如果非null，并且first的hash值为-1，说明其他线程在扩容，参与一起扩容；如果非null，并且hash值不为-1，Synchronized锁住first节点，判断是链表还是红黑树，然后插入
3. get方法。在1.7中，变量value是由volatile修饰，保证在并发情况下，读出来的数据是最新的数据，如果get到的是null值采取加锁。在1.8中类似
4. resize。1.7中，在put操作时做扩容，在获得锁之后，在单线程中去做扩容（1.new个2倍数组2.遍历old数组节点搬去新数组）。在1.8中，扩容支持并发迁移节点，从old数组的尾部开始，如果该桶倍其他线程处理过了，就创建一个ForwardingNode节点放到该桶的首节点，hash值为-1，其他线程判断hash值为-1后就直到该桶被处理过了。
5. 计算size。1.7中先采取不加锁的方式，算两次，如果两次结果一样，说明时正确的，返回；如果不一样，则锁住所有segment，重新计算所有segment的count和。1.8中。用baseCount来记录当前节点的个数。首先尝试通过CAS修改baseCount，如果多线程竞争激烈，某些线程CAS失败，那就CAS尝试将CELLSBUSY置1，成功则把baseCount变化的次数暂存到一个数组countercells，后续数组counterCells的值会加到baseCount中。如果CELLSBUSY置1失败又会反复进行CAS basecount和CAS counterCells数组

```java
/**
put操作
*/
public V put(K key, V value) {
        return putVal(key, value, false);
}
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    // hash码再散列,和HashMap的扰动函数没什么区别，只是额外把位数控制在int最大整数之内
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            // tab初始化
            tab = initTable();
        // 确定此hash所在的bin，如果bin是空的，那么只需要做一次CAS操作即可，无需用锁
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
        }
        // 如果f的hash为-1，则说明f为ForwardingNode节点，意味着有其它线程正在扩容，则帮助一起进行扩容操作
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 采用同步内置锁，保证同一时刻只有一个线程能够修改这个bin
            synchronized (f) {
                // 再次检查，防止被其他线程修改
                if (tabAt(tab, i) == f) {
                    //说明f是链表结构的头结点，遍历链表，如果找到对应的node节点，则修改value，
                    //否则在链表尾部加入节点
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&((ek = e.key) == key ||
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
                    //如果f是TreeBin类型节点，说明f是红黑树根节点，则在树结构上遍历元素，
                    //更新或增加节点。
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //如果链表中节点数binCount >= TREEIFY_THRESHOLD(默认是8)，
            //则把链表转化为红黑树结构。
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

private final void addCount(long x, int check) {
    //....
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}

private final void tryPresize(int size) {
    // c是2的N次幂
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
            tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            // CAS修改sizeClt为-1，表示table正在进行初始化
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    // 确认其他线程没有对table进行修改
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        // 如果扩容大小没有达到阈值，或者超过最大容量
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;
        else if (tab == table) {
            // 生成表的生成戳，每个n都有不同的生成戳
            int rs = resizeStamp(n);
            if (sc < 0) {
                Node<K,V>[] nt;
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
        }
    }
}

```

### ConcurrentHashMap在JDK 1.8中为什么放弃分段锁

主要是为了进一步提高并发性，将锁的级别控制在了更细粒度的table元素级别，也就是说只需要锁住这个链表的head节点，并不会影响其他的table元素的读写，好处在于并发的粒度更细，影响更小。同时分段锁的并发数就是就是分段的个数，基本上是固定的，而JDK 1.8中并发数是和数组的大小相关的。

JDK 1.8中的效率更高的原因：

1. 锁粒度的变小
2. 红黑树的引入对于链表的优化，提高了put和get的效率
3. JVM对synchronized的优化

------

### CopyOnWriteArrayList

### 为什么要使用CopyOnWriteArrayList？

在很多应用场景中，读操作可能会远远大于写操作。由于读操作不会修改原有的数据，所以多个线程同时访问List的内部数据是安全的，因此对于每次读取都进行加锁是一种资源浪费。因此应该允许多个线程同时读取数据

读读共享、写写互斥、读写互斥、写读互斥

### 如何实现

CopyOnWriteArrayList 类的所有可变操作（add，set等等）都是通过创建底层数组的新副本来实现的。当 List 需要被修改的时候，并不修改原有内容，而是对原有数据进行一次复制，将修改的内容写入副本。写完之后，再将修改完的副本替换原来的数据，这样就可以保证写操作不会影响读操作了

读取操作没有任何同步控制和锁操作，因为内部数组 array 不会发生修改，只会被另外一个 array 替换，因此可以保证数据安全

CopyOnWriteArrayList 写入操作 add() 方法在添加集合的时候加了锁，保证了同步，避免了多线程写的时候会 copy 出多个副本出来。

### CopyOnWriteArrayList的问题

1. 内存占用（新写入的对象和旧的对象）
2. 数据一致性（只能保证数据最终一致，不能保证数据的实时一致）.拷贝数组、新增元素都需要时间，所以调用一个 set 操作后，读取到数据可能还是旧的

## ConcurrentLinkedQueue

Java提供的线程安全的 Queue 可以分为阻塞队列和非阻塞队列，其中阻塞队列的典型例子是 BlockingQueue，非阻塞队列的典型例子是ConcurrentLinkedQueue，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。 阻塞队列可以通过加锁来实现，非阻塞队列可以通过 CAS 操作实现。

ConcurrentLinkedQueue使用链表作为其数据结构,主要使用 CAS 非阻塞算法来实现线程安全

## BlockingQueue

所有的阻塞队列：

+ SynchronousQueue： 一个不存储元素的阻塞队列。使用这个队列，意味着提交的任务不会被保存，新任务总是提交给线程执行，若无空闲线程会尝试创建。若达到最大数量线程则执行拒绝策略
+ ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
+ LinkedBlockingQueue ：一个由链表结构组成的有(optionally-bounded)阻塞队列。
+ PriorityBlockingQueue ：一个支持优先级排序的无界阻塞列。
+ DelayQueue： 一个使用优先级队列实现的无界阻塞队列。队列使用 PriorityQueue 来实现，队列中的元素必须实现Delayed接口。在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素
+ LinkedTransferQueue： 一个由链表结构组成的无界阻塞队列。
+ LinkedBlockingDeque： 一个由链表结构组成的双向阻塞队列

ArrayBlockingQueue、LinkedBlockingQueue、PriorityBlockingQueue

+ ArrayBlockingQueue:
  底层采用数组来实现。ArrayBlockingQueue一旦创建，容量不能改变。其并发控制采用可重入锁来控制，不管是插入操作还是读取操作，都需要获取到锁才能进行操作。当队列容量满时，尝试将元素放入队列将导致操作阻塞;尝试从一个空队列中取一个元素也会同样阻塞
  ArrayBlockingQueue 默认情况下不能保证线程访问队列的公平性
+ LinkedBlockingQueue
  底层基于单向链表实现的阻塞队列，可以当做无界队列也可以当做有界队列来使用，同样满足FIFO的特性，与ArrayBlockingQueue 相比起来具有更高的吞吐量，为了防止 LinkedBlockingQueue 容量迅速增，损耗大量内存。通常在创建LinkedBlockingQueue 对象时，会指定其大小，如果未指定，容量等于`Integer.MAX_VALUE`
+ PriorityBlockingQueue :
  支持优先级的无界阻塞队列。默认情况下元素采用自然顺序进行排序，也可以通过自定义类实现 compareTo() 方法来指定元素排序规则，或者初始化时通过构造器参数 Comparator 来指定排序规则
  PriorityBlockingQueue 并发控制采用的是 ReentrantLock，队列为无界队列,是 PriorityQueue 的线程安全版本。不可以插入 null 值

## ConcurrentSkipListMap

其内部采用SkipList数据结构，由于跳表的结构，用跳表实现的Map是有序的

SkipList相比于红黑树，插入和查找的效率一样，但其实现要简单。

在保证并发安全时，采用volatile和CAS算法保证

### ThreadLocal

#### ThreadLocal是什么

从名字我们就可以看到ThreadLocal叫做线程变量，意思是ThreadLocal中填充的变量属于**当前**线程，该变量对其他线程而言是隔离的。ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。

只要线程是活动的并且 ThreadLocal 实例是可访问的，每个线程都会保持对其线程局部变量副本的隐式引用；在线程消失之后，其线程局部实例的所有副本都会被垃圾回收（除非存在对这些副本的其他引用）。

#### 应用场景

**1、在进行对象跨层传递的时候，使用ThreadLocal可以避免多次传递，打破层次间的约束。**

**2、线程间数据隔离**

**3、进行事务操作，用于存储线程事务信息。**

**4、数据库连接，Session会话管理。**

#### 原理分析

​	ThreadLocal中的嵌套内部类ThreadLocalMap，这个类本质上是个map，和HashMap之类的实现相似，依然是key-value的形式，其中有一个内部类Entry，其中key可以看做是ThreadLocal实例，但是其本质是持有ThreadLocal实例的弱引用。

​	在ThreadLocal中并没有对于ThreadLocalMap的引用，是的，ThreadLocalMap的引用在Thread类中，代码如下。每个线程在向ThreadLocal里塞值的时候，其实都是向自己所持有的ThreadLocalMap里塞入数据；读的时候同理，首先从自己线程中取出自己持有的ThreadLocalMap，然后再根据ThreadLocal引用作为key取出value，基于以上描述，ThreadLocal实现了变量的线程隔离。

![](F:\mycode\knowledgeArrangement\web\框架\threadLocal_key.png)

- 首先，主线程定义的两个ThreadLocal变量，和两个子线程——线程A和线程B。
- 线程A和线程B分别持有一个ThreadLocalMap用于保存自己独立的副本，主线程的ThreadLocal中封装了get()和set()之类的方法。
- 在线程A和线程B中调用ThreadLocal的set方法，会首先通过getMap(Thread.currentThread)获得线程A或者是线程B持有的ThreadLocalMap,在调用map.put()方法，并将ThreadLocal作为key。
- get()方法和set()方法原理类似，也是先获取当前调用线程的ThreadLocalMap,再从map中获取value，并将ThreadLocal作为key。