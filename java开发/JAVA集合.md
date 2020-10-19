[TOC]

# Java集合

集合主要包括Collection和Map两种，Collection存储对象的集合，而Map存储键值对的映射表

![Collection](F:/mycode/知识点整理/pics/Collection继承关系.png)

1. Set

   + TreeSet：基于**红黑树**实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。

   + HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。

   + LinkedHashSet：具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。

2. List

   + ArrayList：基于动态数组实现，支持随机访问。
   + Vector：和 ArrayList 类似，但它是线程安全的。
   + Stack：继承自Vector，所以也是线程安全的
   + LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

3. Queue

   + LinkedList：可以用它来实现双向队列。
   + PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

4. Map

   + TreeMap：基于红黑树实现。
   + HashMap：基于哈希表实现。
   + HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。
   + LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

   ![Map](F:/mycode/知识点整理/pics/Map继承关系.png)

## fail-fast 机制

fail-fast 机制，即快速失败机制，是java集合中的一种错误检测机制。当==在迭代==集合的==过程中==该==集合在结构上发生改变==的时候，就有可能会发生fail-fast，即==抛出ConcurrentModificationException异常==。

例如：假设线程A通过 Iterator 在遍历集合中的元素，在某个时候线程 B 修改了集合的结构（是结构上面的修改，而不是简单的修改集合元素的内容）， 那么这个时候程序就会抛出ConcurrentModificationException 异常， 从而产生 fail-fast 机制。

产生的原因：
当调用容器的 iterator() 方法返回 Iterater 对象时， 对容器结构修改的次数modCount赋值给了expectedModCount变量,在调用 next() 方法时， 会比较 expectedModCount 与容器对象维护的实际修改次数是否相等， 若二者不相等， 则会抛出ConcurrentModificationException 异常。

解决办法：
1.如果在遍历集合的同时， 需要删除元素的话， 可以用 iterator 里面的 remove()方法删除元素，但是该方法不能指定元素只能删除当前遍历过的元素；
2、可以采用并发包中的类来代替

注意：java.util包下的集合类都是快速失败机制的, 不能在多线程下发生并发修改(迭代过程中被修改).

## fail-safe机制

采用安全失败机制的集合容器,在遍历时不是直接在集合内容上访问的,而是==先copy原有集合内容,在拷贝的集合上进行遍历==。

原理：由于迭代时是对原集合的拷贝的值进行遍历,所以在遍历过程中对原集合所作的修改并不能被迭代器检测到,所以不会触发ConcurrentModificationException

缺点：基于拷贝内容的优点是避免了ConcurrentModificationException,但同样地, 迭代器并不能访问到修改后的内容 (简单来说就是, 迭代器遍历的是开始遍历那一刻拿到的集合拷贝,在遍历期间原集合发生的修改迭代器是不知道的)

注意：java.util.concurrent包下的容器都是安全失败的,可以在多线程下并发使用,并发修改.

## List、Map和Set的区别

+ List
  1. 允许重复的对象
  2. 可以插入多个null元素
  3. 保持插入顺序

+ Set
  1. 不允许重复对象
  2. 无序容器，除了TreeSet通过Comparator或者Comparable维护了一个排序顺序
  3. 只允许一个null元素

+ Map
  1. List、Set实现的是collection接口，Map实现的是Map接口
  2. 存储的每个Entry有key和value两个对象，key是唯一的
  3. 可以拥有随意个null值，但是只能有一个null键

## 红黑树

### 红黑树的特点

红黑树是一种自平衡的**二叉查找树**，是一种高效的查找树。红黑树具有良好的效率，可在 $O(logN)$ 时间内完成查找。

Java 中的 TreeMap，JDK 1.8 中的 HashMap、C++ STL 中的 map 均是基于红黑树结构实现的

普通的二叉查找树在极端情况下可退化成链表，此时的增删查效率都会比较低下。为了避免这种情况，就出现了一些自平衡的查找树，比如 AVL，红黑树等。这些自平衡的查找树通过定义一些性质，将任意节点的左右子树高度差控制在规定范围内，以达到平衡状态

![红黑树](F:/mycode/知识点整理/pics/红黑树.jpg)

红黑树的性质：

+ 节点是红色或黑色。
+ 根是黑色。
+ 所有叶子都是黑色（叶子是NIL节点）。
+ 每个红色节点必须有两个黑色的子节点。从每个叶子到根的所有路径上不能有两个连续的红色节点。
+ 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点（简称黑高）。

任意节点到其每个叶子节点路径最长不会超过最短路径的2倍

当某条路径最短时，这条路径必然都是由黑色节点构成。当某条路径长度最长时，这条路径必然是由红色和黑色节点相间构成（性质4限定了不能出现两个连续的红色节点）。而性质5又限定了从任一节点到其每个叶子节点的所有路径必须包含相同数量的黑色节点。此时，在路径最长的情况下，路径上红色节点数量 = 黑色节点数量。该路径长度为两倍黑色节点数量，也就是最短路径长度的2倍

参考：

+ <http://www.tianxiaobo.com/2018/01/11/%E7%BA%A2%E9%BB%91%E6%A0%91%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90/>

### 红黑树的操作

红黑树的插入过程和二叉查找树插入过程基本类似，不同的地方在于，红黑树插入新节点后，需要进行调整，以满足红黑树的性质。

在插入或删除时采用旋转和变色的方式来维保证红黑树的性质。

在插入新节点时，这个节点是红色的原因：

如果插入的节点是黑色，那么这个节点所在路径比其他路径多出一个黑色节点，这个调整起来会比较麻烦（参考红黑树的删除操作，就知道为啥多一个或少一个黑色节点时，调整起来这么麻烦了）。如果插入的节点是红色，此时所有路径上的黑色节点数量不变，仅可能会出现两个连续的红色节点的情况。这种情况下，通过变色和旋转进行调整即可。

#### 插入操作

如果插入节点的父亲是黑色，则直接插入即可

新插入的节点的父节点都是红色的

1. 叔叔节点也为红色.父节点和叔叔节点与祖父节点的颜色互换。下图中如果A不是黑色则继续修复
   ![1](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/1a130fa3.png)
2. 叔叔节点为空，且祖父节点、父节点和新节点处于一条斜线上。则将B节点进行右旋操作，并且和父节点A互换颜色
   ![2](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/d04da718.png)
3. 叔叔节点为空，且祖父节点、父节点和新节点不处于一条斜线上。先对C左旋
   ![3](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/f1098971.png)

#### 删除操作

如果是叶子节点就直接删除，如果是非叶子节点，会用对应的中序遍历的后继节点来顶替要删除节点的位置.(也就是右子树的最左结点)

删除后就需要做删除修复操作，使的树符合红黑树的定义，符合定义的红黑树高度是平衡的

红黑树的删除操作是最复杂的操作，复杂的地方就在于当删除了黑色节点的时候，如何从兄弟节点去借调节点，以保证树的颜色符合定义。由于红色的兄弟节点是没法借调出黑节点的，这样只能通过选择操作让他上升到父节点，而由于它是红节点，所以它的子节点就是黑的，可以借调。

对于兄弟节点是黑色节点的可以分成3种情况来处理，

+ 当所有的兄弟节点的子节点都是黑色节点时，可以直接将兄弟节点变红，这样局部的红黑树颜色是符合定义的。但是整颗树不一定是符合红黑树定义的，需要往上追溯继续调整。
+ 对于兄弟节点的子节点为左红右黑或者 (全部为红，右红左黑)这两种情况，可以先将前面的情况通过选择转换为后一种情况，在后一种情况下，因为兄弟节点为黑，兄弟节点的右节点为红，可以借调出两个节点出来做黑节点，这样就可以保证删除了黑节点，整棵树还是符合红黑树的定义的，因为黑色节点的个数没有改变。
+ 红黑树的删除操作是遇到删除的节点为红色，或者追溯调整到了root节点，这时删除的修复操作完毕

参考：

+ <https://tech.meituan.com/2016/12/02/redblack-tree.html>

## List

### Arraylist 与 LinkedList 异同

1. 是否保证线程安全： ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；
2. 底层数据结构： Arraylist 底层使用的是Object数组；LinkedList 底层使用的是双向链表数据结构（JDK1.6之前为循环链表，JDK1.7取消了循环。注意双向链表和双向循环链表的区别：）； 详细可阅读JDK1.7-LinkedList循环链表优化
3. 插入和删除是否受元素位置的影响：
   1. ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行add(E e) 方法的时候， ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是O(1)。但是如果要在指定位置 i 插入和删除元素的话（add(int index, E element) ）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。
   2. LinkedList 采用链表存储，所以插入，删除元素时间复杂度不受元素位置的影响，都是近似 O（1）而数组为近似 O（n）。
4. 是否支持快速随机访问： LinkedList 不支持高效的随机元素访问，而 ArrayList 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index) 方法)。
5. 内存空间占用： ArrayList的空间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。

ArrayList适用于读多写少。

LinkedList适用于写多读少。

### ArrayList 与 Vector 区别

* ArrayList 和 Vector 都是用数组实现的
* Vector类的所有方法都是同步的,使用synchronize，而Arraylist不是同步的
* Vector扩容时变为原容量的2倍，而ArrayList扩为1.5倍

### ArrayList是如何扩容的

ArrayList是基于数组实现的，如果未指定初始容量，在**插入第1个元素**时会默认分配10个对象空间。在插入新元素时，当元素的个数大于容量时，扩容为原数组容量的`1.5`倍，然后将原数组中的元素全部copy到新数组中(调用Arrays.copyOf)，接着再往数组末尾添加元素。若指定初始容量，则按初始容量分配空间。该扩容和初始化操作主要由grow方法完成。

如果ArrayList存储比较多的数据，最好指定初始容量，防止copy的性能损耗

### SynchronizedList与Vector的区别

==SynchronizedList是Collections的静态内部类==。两者本质上都是通过给synchorized实现的，区别主要在于ArrayList与Vector的扩容速度不同，需要线程安全时，如果增量比较快，应该使用Vector

### Arrays.asList使用时需要注意什么

该方法时Arrays中的静态方法，可以将数组转为集合。

* 该方法不能作用于基本类型的数组
* Arrays.asList返回的ArrayList不是java.util下的ArrayList，而是在Arrays中的静态内部类，它继承了AbstractList类，但没有重写add，romve等方法，所以Arrays.asList返回的list不支持修改操作

### RandomAccess接口

RandomAccess 接口是一个标识, 使实现这个接口的类具有随机访问功能

实现了RandomAccess接口的list，优先选择普通for循环 ，其次foreach,

未实现RandomAccess接口的ist， 优先选择iterator遍历（foreach遍历底层也是通过iterator实现的），大size的数据，千万不要使用普通for循环

### forEach的性能讨论

foreach用于链表结构存储访问速度非常快，比for快；用于ArrayList则比for慢。for循环是根据下标一个个检索获取，而foreach是通过迭代器Iterator，不断获取next元素。

### 如何实现数组与List的相互转换

List转数组：toArray(arraylist.size())方法；数组转List:Arrays的asList(a)方法

```java
List<String> arrayList = new ArrayList<String>();
arrayList.add("s");
arrayList.add("e");
arrayList.add("n");
/**
 * ArrayList转数组
 */
int size=arrayList.size();
String[] a = arrayList.toArray(new String[size]);
//输出第二个元素
System.out.println(a[1]);//结果：e
//输出整个数组
System.out.println(Arrays.toString(a));//结果：[s, e, n]
/**
 * 数组转list
 */
List<String> list=Arrays.asList(a);
/**
 * list转Arraylist
 */
List<String> arrayList2 = new ArrayList<String>();
arrayList2.addAll(list);
System.out.println(list);
```

### 如何求ArrayList集合的交集 并集 差集 去重复并集

需要用到List接口中定义的几个方法：

* addAll(Collection<? extends E> c) :按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾
* retainAll(Collection<?> c): 仅保留此列表中包含在指定集合中的元素。
* removeAll(Collection<?> c) :从此列表中删除指定集合中包含的所有元素。

```java
List<Integer> list1 = new ArrayList<Integer>();
list1.addAll(Arrays.asList(1,2,3,4));
List<Integer> list2 = new ArrayList<Integer>();
list2.addAll(Arrays.asList(2,3,4,5));
// 并集
list1.addAll(list2);

// 交集
list1.retainAll(list2);

// 差集
list1.removeAll(list2);

// 无重复并集
list2.removeAll(list1);
list1.addAll(list2);
```

### LinkedList的get方法

get方法的参数是索引，在get时，会判断index属于链表的前半部分还是后半部分，如果是后半部分会从尾部开始遍历

## Map

### HashMap的底层实现

+ capacity：容量，数组大小,默认16。
+ loadFactor：负载因子，默认0.75。
+ TREEIFY_THRESHOLD：默认为8。决定采用树还是列表结构维护一个bin，当同一个bin中节点数大于该值时,bin由list转为tree。该值必须大于2并且至少为8
+ UNTREEIFY_THRESHOLD：默认为6。在resize操作是将tree转化（分割）为list的域置，应该小于TREEIFY_THRESHOLD，至少为6
+ 阈值：threshold = capacity * loadFactor，当前 HashMap 所能容纳键值对数量的最大值
+ MIN_TREEIFY_CAPACITY：允许bin被转换为tree的最小table容量（如果太多node在一个bin中，table将被扩容）

扰动函数：指 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法， 使用扰动函数之后可以减少碰撞。

1. JDK1.8之前

   JDK1.8 之前 HashMap 底层是 数组和链表 结合在一起使用也就是 链表散列。HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

   **拉链法：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可**

2. JDK1.8之后

   JDK1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间

注意：jdk1.7中节点是插入链表头部的，jdk1.8中节点是插入列表尾部的

### hashMap的扰动函数

```Java
	static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

把哈希值右移16位，也就正好是自己长度的一半，之后与原哈希值做异或运算，这样就混合了原哈希值中的高位和低位，增大了**「随机性」**。使用扰动函数就是为了增加随机性，让数据元素更加均衡的散列

### hashMap的resize函数

获取旧table的容量，如果table尚未初始化则为0。如果旧table的容量大于等于最大容量(MAXINUM_CAPACITY)，把阈值设为最大整数后返回旧table，不扩容。如果新容量（为旧容量的二倍）小于最大容量并且旧容量大于等于默认初始容量（默认16），将容量和阈值都扩大2倍。如果旧容量小于等于0但阈值大于0，将新容量设为阈值大小。否则将新容量设置为默认初始容量（DEFAULT_INITIAL_CAPACITY），新阈值设为默认负载因子(DEFAULT_LOAD_CAPACITY)乘以默认初始容量。接下来如果新阈值还是0，就将其设为新容量*负载因子（loadFactory），如果新容量查过最大容量九江阈值设为最大整数。创建新的table数组（Node数组，大小为之前设置的新容量）。如果旧table为空，那就初始化table，此时就结束了。否则开始扩容。

##### 扩容部分

遍历旧table中的每一个Node对象，假定当前下标为j。将旧table当前位置的Node引用置为null。如果该Node对象不为空，首先检查其next属性是否为空，如果是直接通过e.hash&（newCap-1）确定其在新table中的下标，并完成赋值。如果这个e是TreeNode的对象（TreeNode类也是Node类的子类），说明这个节点是个红黑树的结构，进行树的拆分。接下来e只能是个链表结构了，拆分链表，判断原链表中当前节点该分给那个子链表是根据e.hash & oldCap == 0来判断的。如果`e.hash & oldCap=0`,拆分后一个放在新table的的j下标;如果`e.hash & oldCap`不为0,一个放在j+oldCap处。

### hashmap的put过程

put过程是通过`putVal()`完成，首先判断桶数组table是否初始化，因为桶数组是被**延迟到插入新数据时再进行初始化**，未初始化则调用`resize()`方法进行初始化。然后判断桶中（(cap-1)&hash）是否包含键值对节点引用，不包含则将新键值对节点的引用存入桶中即可。如果桶中已经存在节点，判断键的值以及节点 hash 是否等于链表中的第一个键值对节点时，则如果相等则将 e 指向该键值对。否则需要将节点存入红黑树或者如果桶中的引用类型为 TreeNode，则调用红黑树的插入方法。如果是链表，采用遍历链表的方式插入，如果链表中不包含要插入的节点，则将节点放到链表的最后。在插入后如果链表结点数量达到8则需要树化。

> 1)对 Key 求 Hash 值，然后再计算下标
> 2)如果没有碰撞，直接放入桶中（碰撞的意思是计算得到的 Hash 值相同，需要放到同一个 bucket 中）
> 3)如果碰撞了，以链表的方式链接到后面
> 4)如果链表长度超过阀值（TREEIFY THRESHOLD==8），就把链表转成红黑树，链表长度低于6，就把红黑树转回链表
> 5)如果节点已经存在就替换旧值
> 6)如果桶满了（容量16*加载因子0.75），就需要 resize（扩容2倍后重排）

插入后如果键值对的数量超过阈值时，则进行扩容

### HashMap的get流程，get的复杂度

get是通过`getNode()`实现的，首先通key的hash值找到在桶数组中的位置(由于桶数组的大小总是2的幂，此时，`(n - 1) & hash` 等价于对n取余)。

如果该位置有元素，则比较该数组的元素是否是要找的key，如果是则返回元素。不是则需要进一步查找，需要判断结点是否是TreeNode类型，是则通过红黑树查找，否则遍历链表

### hashmap是如何解决冲突的？

1.8之前：链地址法，即数组+链表的方式

1.8之后：数组+链表+红黑树

### HashMap扩容过程，如何定位值在新数组中的位置

如果桶数组中只有一个元素是通过`hash&(newCap-1)`计算出来的。
有多个

将节点加到链表后容量扩充为原来的两倍，然后对每个节点重新计算哈希值
这个值只可能在两个地方：一种是在原下标位置，另一种是在下标为<原下标+原容量>的位置

扩容过程：

+ 先判断table是否为空，不为空判断table容量是否超过容量最大值，如果超过则不再扩容，否则旧容量和阈值的2倍计算新容量和阈值的大小。
+ 如果table为空则表明table未初始化，此时需要根据调用的构造函数来确定容量和阈值。
  + 如果在构造时传入了初始容量和负载因子，则将容量赋为阈值threshold，因为在构造时threshold暂时存储了初始容量。新的阈值是通过容量*负载因子计算出来的。若
  + 若采用默认构造建立对象，则容量是默认初始容量16，阈值根据初始容量*默认负载因子0.75计算出来
+ 计算出新的容量和阈值，新建一个Node数组用于存储元素
+ 遍历原桶数组，如果数组元素不为空则需要将原数组的元素放到新的桶数组中
  + 如果该位置只有一个结点，则通过`hash&(newCap-1)`计算出新的位置并且放入元素
  + 如果是树结点，则需要对红黑树进行分裂
  + 如果是链表的情况，则分裂链表
    + 如果`e.hash & oldCap=0`，则表明这个结点在新数组的索引和原数组是一样的
    + 如果`e.hash & oldCap`不为0，则表进结点在新数组的索引是`oldCap+原索引`，原数组大小+原索引

对于扩容为什么是2倍，因为HashMap中数组的容量是2的N次方，所以扩容完成后也应该是2的次方

### 为什么HashMap中String、Integer这样的包装类适合作为key

+ String、Integer等包装类具有不可变性，保证了key的不可更改性，不会出现放入和获取时哈希码不同的情况。同时String的hashcode是缓存的，不用重复计算
+ 内部重写了equals、hashcode等方法不容易出现hash值的计算错误

参考：<https://juejin.im/post/5aa5d8d26fb9a028d2079264>

### HashMap 的长度为什么是2的幂次方

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash 值的范围值-2147483648到2147483647，前后加起来大概42亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ (n - 1) & hash ”。（n代表数组长度）。这也就解释了 HashMap 的长度为什么是2的幂次方。

这个算法应该如何设计呢？

我们首先可能会想到采用%取余的操作来实现。但是，重点来了：“取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说 `hash%length==hash&(length-1)`的前提是 length 是2的 n 次方；）。” 并且 采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方

### HashMap为什么是线程不安全的

在HashMap中的容量超过设定的阈值时会进行扩容，扩容时需要将已有的元素进行rehash，在rehash的过程中可能出现环形链表的问题，这个问题在JDK 1.8中已经被解决。

同时还存在着数据丢失等问题，而且记录元素个数的变量size也不是volatile，多线程情况下会出现问题。

数据丢失问题：在put操作时，会判断数组中元素是否为null，如果为null会新建一个结点放入数组中，如果两个线程都进入了if语句块中，都会创建一个结点放入数组，这样会出现一个结点的引用覆盖，导致数据丢失。在put操作时，在遍历链表时找到插入位置，可能也会出两个线程在同一个位置进行插入的情况

```java
//JDK 1.8中的put源码的一部分
if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
```

补充内容：环形链表的形成：<https://www.toutiao.com/i6545790064104833539/>

### HashMap为什么采用尾插法

HashMap在jdk1.7中采用头插入法，在扩容时会改变链表中元素原本的顺序，以至于在并发场景下导致链表成环的问题。而在jdk1.8中采用尾插入法，在扩容时会保持链表元素原本的顺序，就不会出现链表成环的问题了

### HashMap的扰动函数（也就是hash()）

HashMap的扰动函数也就是hash方法的目的就是减少碰撞（自己实现的hashCode方法太差，或者因为除留余数法忽略高位信息导致的碰撞）。

jdk1.8的hash方法(高16位不变，低16位与高16位异或)：

```java
 static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^ ：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  } 
```

不直接使用key hash值的原因与HashMap中table下标的计算有关。

```java
n = table.length;
index = （n-1） & hash;
```

因为，table的长度都是2的幂，因此index仅与hash值的低logn位有关，hash值的高位都被与操作置为0了。 
假设table.length=2^4=16。
![20190211145854.png](https://raw.githubusercontent.com/shaxinlei/MarkdownImages/master/20190211145854.png)
由上图可以看到，只有hash值的低4位参与了运算。 
这样做很容易产生碰撞。设计者权衡了speed, utility, and quality，将高16位与低16位异或(混合高低位信息，保留高位特征)来减少这种影响。设计者考虑到现在的hashCode分布的已经很不错了，而且当发生较大碰撞时也用树形存储降低了冲突。仅仅异或一下，既减少了系统的开销，也不会造成的因为高位没有参与下标的计算(table长度比较小时)，从而引起的碰撞。

### HashMap 和 Hashtable 的区别

+ 线程是否安全： HashMap 是非线程安全的，HashTable 是线程安全的；HashTable 内部的方法基本都经过 synchronized 修饰。（如果你要保证线程安全的话就使用 ConcurrentHashMap 吧！）；
+ 效率： 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。另外，HashTable 基本被淘汰，不要在代码中使用它；
+ 对Null key 和Null value的支持： HashMap中，null 可以作为键，这样的键只有一个，值也可以为null。而HashTable不能用null作为键，使用会抛出NullPointerException。
+ 初始容量大小和每次扩充容量大小的不同 ：
  + Hashtable 默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。
  + Hashtable创建时如果给定了容量初始值，那么 Hashtable 直接使用给定的大小，而 HashMap 会将其扩充为2的幂次方大小（HashMap 中的tableSizeFor()方法保证）。也就是说 HashMap 总是使用2的幂作为哈希表的大小
+ 底层数据结构： JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制

### HashTable 的效率比较低的原因

因为当一个线程访问HashTable的同步方法时，访问其他同步方法的线程就可能会进入阻塞或者轮询状态。当线程A使用put进行添加元素，线程B不但不能用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争激烈效率越低。

### TreeMap

TreeMap是一个有序的key-value集合，能够把保存的记录根据键排序，默认是按键值的升序排序，也可以指定比较器

TreeMap 是通过实现 SortMap 接口，能够把它保存的键值对根据 key 排序，基于红黑树，从而保证 TreeMap 中所有键值对处于有序状态。

由于底层是红黑树，所以可以在$O(logn)$复杂度内完成操作。

TreeMap在构造时要么传入一个Comparator对象，用于key的比较，要么key自身实现了Comparable接口

### LinkedHashMap是如何保证有序的

LinkedHashMap 继承自 HashMap，所以底层仍是基于拉链式散列结构。该结构由数组和链表或红黑树组成

LinkedHashMap 在上面结构的基础上，增加了一条双向链表，从而可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑

![LinkedHashMap](F:/mycode/知识点整理/pics/LinkedHashMap.jpg)

LinkedHashMap可以保证访问排序或插入排序，通过构造函数可以设置（默认插入排序）

* 插入排序：put的时候的顺序是什么，取出来的时候就是什么样子
  插入排序主要通过LinkedhashMap中的afterNodeInsertion方法实现
* 访问排序：get的时候，会改变元素的顺序，会把该元素移到数据的末尾（LRU利用LinkedHashMap实现时就会开启访问排序）
  访问排序主要通过LinkedhashMap中的afterNodeAccess方法实现

## Set

### Set的实现

Set主要有TreeSet，HashSet，LinkedHashSet，底层分别是用TreeMap、HashMap、LinkedHashMap实现的。key是Set存储的元素，value是一个静态的Object，所有的key对应同一个Object

HashSet是非线程安全的，允许null值。HashSet的值是存在HashMap的key上面的，而value是一个统一值。添加值得时候会先获取对象的hashCode方法，如果hashCode 方法返回的值一致，则再调用equals方法判断是否一致，如果不一致才add元素。

LinkedHashSet维护插入的顺序。

### TreeSet和HashSet的区别

1. TreeSet 是二叉树实现的，Treeset中的数据是自动排好序的，不允许放入null值 。
2. HashSet 是哈希表实现的，HashSet中的数据是无序的，可以放入null，但只能放入一个null，两者中的值都不能重复。
3. HashSet要求放入的对象必须实现HashCode()方法，放入的对象，是以hashcode码作为标识的，而具有相同内容的String对象，hashcode是一样，所以放入的内容不能重复。但是同一个类的对象可以放入不同的实例。

适用场景分析：

HashSet是基于Hash算法实现的，其性能通常都优于TreeSet。我们通常都应该使用HashSet，在我们需要排序的功能时，我们才使用TreeSet

### Set如何保证不重复

HashSet的内部实现为HashMap，TreeSet的内部实现为TreeMap，LinkedHashSet的内部实现为LinkedHashMap

### HashSet如何检查重复

当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals()方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。

hashCode()与equals()的相关规定：

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个equals方法返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. 综上，equals方法被覆盖过，则hashCode方法也必须被覆盖
5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。

## 杂

### 集合类的常用方法

* Collection常用方法
  * add(E e) ，添加元素
  * clear() ，暴力清除集合中所有元素
  * contains(Object o)， 返回值类型：boolean。判断集合是否包含某个元素
  * isEmpty() ，返回值类型：boolean。如果此集合不包含元素，则返回true。
  * iterator() 迭代器。返回值类型：Iterator
  * size() 返回值类型：int。返回集合中的元素数
* List特有且常用的方法
  * add(int index,Object element)：在指定位置添加元素
  * get(int index)：获取指定位置的元素
  * remove(int index)：根据索引删除元素，返回被删除的元素
  * set(int index,Object element)：根据索引修改元素，返回被修改的元素
* LinkedList特有的方法
  * addFirst(Object e)
  * addLast(Object e) //和add()功能一样，所以不常用此方法
  * getFirst()
  * getLast()
  * removeFirst()
  * removeLast()
* Map集合的常用方法
  * put(K key, V value)
  * get(Object key)
  * putAll(Map<? extends K,? extends V> m)：向map集合中添加指定集合的所有元素
  * clear()：把map集合中所有的键值删除
  * containsKey(Object key)
  * containsValue(Object value)
  * entrySet()
  * keySet()
  * equals(Object o)：判断两个Set集合的元素是否相同
  * isEmpty()
  * remove(Object key)
  * size()
  * Collection<V> values()

### Java 迭代器

**迭代器模式：就是提供一种方法对一个容器对象中的各个元素进行访问，而又不暴露该对象容器的内部细节。**

### 概述

　　Java集合框架的集合类，我们有时候称之为容器。容器的种类有很多种，比如ArrayList、LinkedList、HashSet...，每种容器都有自己的特点，ArrayList底层维护的是一个数组；LinkedList是链表结构的；HashSet依赖的是哈希表，每种容器都有自己特有的数据结构。

　　因为容器的内部结构不同，很多时候可能不知道该怎样去遍历一个容器中的元素。所以为了使对容器内元素的操作更为简单，Java引入了迭代器模式！ 

　　把访问逻辑从不同类型的集合类中抽取出来，从而避免向外部暴露集合的内部结构。

对于这两种方式，我们总是都知道它的内部结构，访问代码和集合本身是紧密耦合的，无法将访问逻辑从集合类和客户端代码中分离出来。不同的集合会对应不同的遍历方法，客户端代码无法复用。在实际应用中如何将上面两个集合整合是相当麻烦的。所以才有Iterator，它总是用同一种逻辑来遍历集合。使得客户端自身不需要来维护集合的内部结构，所有的内部状态都由Iterator来维护。客户端不用直接和集合进行打交道，而是控制Iterator向它发送向前向后的指令，就可以遍历集合。

**java.util.Iterator**

在Java中Iterator为一个接口，它只提供了迭代的基本规则。在JDK中它是这样定义的：对Collection进行迭代的迭代器。迭代器取代了Java Collection Framework中的Enumeration。迭代器与枚举有两点不同:

　　1. 迭代器在迭代期间可以从集合中移除元素。

　　2. 方法名得到了改进，Enumeration的方法名称都比较长。

```java
package java.util;
public interface Iterator<E> {
    boolean hasNext();//判断是否存在下一个对象元素

    E next();//获取下一个元素

    void remove();//移除元素
}
```

**Iterable**

​	Java中还提供了一个Iterable接口，Iterable接口实现后的功能是‘返回’一个迭代器，我们常用的实现了该接口的子接口有:Collection<E>、List<E>、Set<E>等。该接口的iterator()方法返回一个标准的Iterator实现。实现Iterable接口允许对象成为Foreach语句的目标。就可以通过foreach语句来遍历你的底层序列。

　　Iterable接口包含一个能产生Iterator对象的方法，并且Iterable被foreach用来在序列中移动。因此如果创建了实现Iterable接口的类，都可以将它用于foreach中。

```java
Package java.lang;

import java.util.Iterator;
public interface Iterable<T> {
    Iterator<T> iterator();
}
```

**Iterator遍历时不可以删除集合中的元素问题**

在使用Iterator的时候禁止对所遍历的容器进行改变其大小结构的操作。例如: 在使用Iterator进行迭代时，如果对集合进行了add、remove操作就会出现ConcurrentModificationException异常。

因为在你迭代之前，迭代器已经被通过list.itertor()创建出来了，如果在迭代的过程中，又对list进行了改变其容器大小的操作，那么Java就会给出异常。因为此时Iterator对象已经无法主动同步list做出的改变，Java会认为你做出这样的操作是线程不安全的，就会给出善意的提醒（抛出ConcurrentModificationException异常）