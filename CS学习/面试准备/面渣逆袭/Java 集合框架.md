### part 6 说说有哪些常见的集合框架?
1. Java 集合框架可以分为两条大的支线：

①、Collection，主要由 List、Set、Queue 组成：

- List 代表有序、可重复的集合，典型代表就是封装了动态数组的 [ArrayList](https://javabetter.cn/collection/arraylist.html) 和封装了链表的 [LinkedList](https://javabetter.cn/collection/linkedlist.html)；
- Set 代表无序、不可重复的集合，典型代表就是 HashSet 和 TreeSet；
- Queue 代表队列，典型代表就是双端队列 [ArrayDeque](https://javabetter.cn/collection/arraydeque.html)，以及优先级队列 [PriorityQueue](https://javabetter.cn/collection/PriorityQueue.html)。

②、Map，代表键值对的集合，典型代表就是 [HashMap](https://javabetter.cn/collection/hashmap.html)。

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/collection/gailan-01.png)

概览图说明：

①、Collection 接口：最基本的集合框架表示方式，提供了添加、删除、清空等基本操作，它主要有三个子接口：
- `List`：一个有序的集合，可以包含重复的元素。实现类包括 ArrayList、LinkedList 等。
- `Set`：一个不包含重复元素的集合。实现类包括 HashSet、LinkedHashSet、TreeSet 等。
- `Queue`：一个用于保持元素队列的集合。实现类包括 PriorityQueue、ArrayDeque 等。

②、`Map` 接口：表示键值对的集合，一个键映射到一个值。键不能重复，每个键只能对应一个值。Map 接口的实现类包括 HashMap、LinkedHashMap、TreeMap 等。
集合框架位于 java.util 包下，该包含提供了两个常用的工具类：
- [Collections](https://javabetter.cn/common-tool/collections.html)：提供了一些对集合进行排序、二分查找、同步的静态方法。
- [Arrays](https://javabetter.cn/common-tool/arrays.html)：提供了一些对数组进行排序、打印、和 List 进行转换的静态方法。

2. 简单介绍一下队列 Queue
Java 中的队列主要通过 java.util.Queue 接口和 java.util.concurrent.BlockingQueue 两个接口来实现。
PriorityQueue 是一个基于优先级堆的无界队列，它的元素按照自然顺序排序或者 Comparator 进行排序。

ArrayDeque 是一个基于数组的双端队列，可以在两端插入和删除元素。

还有一个大家可能忽略的队列，那就是 LinkedList，它既可以当作 List 使用，也可以当作 Queue 使用。

3. 用过哪些集合类，它们的优劣？
在 Java 中，常用的集合类有 ArrayList、LinkedList、HashMap、LinkedHashMap 等。
ArrayList：ArrayList 可以看作是一个动态数组，它可以在运行时动态扩容。优点是访问速度快，可以通过索引直接查到元素。缺点是插入和删除元素可能需要移动元素，效率就会降低。
LinkedList：LinkedList 是一个双向链表，它适合频繁的插入和删除操作。优点是插入和删除元素的时候只需要改变节点的前后指针，缺点是访问元素时需要遍历链表。
HashMap：HashMap 是一个基于哈希表的键值对集合。优点是插入、删除和查找元素的速度都很快。缺点是它不保留键值对的插入顺序。
LinkedHashMap：LinkedHashMap 在 HashMap 的基础上增加了一个双向链表来保留键值对的插入顺序。

4. 队列的栈区别了解吗
队列是一种先进先出的数据结构, 常用于处理按顺序来的任务
栈是是一种后进先出的数据结构, 这种特性使得栈非常适合于那些需要访问最新添加的数据元素的场合。

5. 哪些是线程安全的?
像 Vector、Hashtable、ConcurrentHashMap、CopyOnWriteArrayList、ConcurrentLinkedQueue、ArrayBlockingQueue、LinkedBlockingQueue 这些都是线程安全的。

6. Java 集合用过哪些？Collection 继承了哪些接口？
最常用的就是封装了动态数组的 ArrayList 和 封装了链表的 LinkedList; 以及键值对 HashMap

Collection 继承了 Iterable 接口，这意味着所有实现 Collection 接口的类都必须实现 `iterator()` 方法，之后就可以使用增强型 for 循环遍历集合中的元素了。


7. ArrayList 和 LickedList 有什么区别
ArrayList 和 LinkedList 的区别主要体现在数据结构、用途、是否支持随机访问、内存占用等方面。

数据结构有什么不同?
ArrayList 基于数组实现
LinkedList 基于链表实现

用途有什么不同?
多数情况下, ArrayList 更方便查找, LinkedList更方便增删

是否支持随机访问?
ArrayList 是基于数组的，也实现了 RandomAccess 接口，所以它支持随机访问，可以通过下标直接获取元素。

内存占用有何不同?
ArrayList 是基于数组的，是一块连续的内存空间，所以它的内存占用是比较紧凑的；但如果涉及到扩容，就会重新分配内存，空间是原来的 1.5 倍，存在一定的空间浪费。

LinkedList 是基于链表的，每个节点都有一个指向下一个节点和上一个节点的引用，于是每个节点占用的内存空间稍微大一点。 

使用 场景有什么不用?
ArrayList 适用于:
- 随机访问频繁的场景
- 读取操作多于增删操作的场景
- 频繁在末尾添加元素的场景

LinkedList 适用于:
频繁插入和删除: \
- 频繁插入的删除的场景
- 顺序访问多于随机访问的场景。
- 队列和栈：由于其双向链表的特性，LinkedList 可以高效地实现队列（FIFO）和栈（LIFO）。

8. ArrayList 的扩容机制了解吗?
ArrayList 确切地说，应该叫做动态数组，因为它的底层是通过数组来实现的，当往 ArrayList 中添加元素时，会先检查是否需要扩容，如果当前容量+1 超过数组长度，就会进行扩容。

9. ArrayList 怎么序列化? 为什么用 transient 修饰数组
ArrayList 的序列化不太一样，它使用`transient`修饰存储元素的`elementData`的数组，`transient`关键字的作用是让被修饰的成员属性不被序列化。

为什么 ArrayList 不直接序列化元素数组呢?
出于效率的考虑，数组可能长度 100，但实际只用了 50，剩下的 50 不用其实不用序列化，这样可以提高序列化和反序列化的效率，还可以节省内存空间。

那么 ArrayList 怎么实现序列化呢?
ArrayList 通过两个方法**readObject、writeObject**自定义序列化和反序列化策略，实际直接使用两个流`ObjectOutputStream`和`ObjectInputStream`来进行序列化和反序列化。


10. 快速失败 和 安全失败了解吗?
**快速失败（fail—fast）**：快速失败是 Java 集合的一种错误检测机制
- 在用迭代器遍历一个集合对象时，如果线程 A 遍历过程中，线程 B 对集合对象的内容进行了修改（增加、删除、修改），则会抛出 Concurrent Modification Exception。
- 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 `modCount` 变量。集合在被遍历期间如果内容发生变化，就会改变`modCount`的值。每当迭代器使用 hashNext()/next()遍历下一个元素之前，都会检测 modCount 变量是否为 expectedmodCount 值，是的话就返回遍历；否则抛出异常，终止遍历。
- 注意：这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改 modCount 值刚好又设置为了 expectedmodCount 值，则异常不会抛出。因此，`不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的 bug。`
- 场景：java.util 包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改），比如 ArrayList 类。\

**安全失败（fail—safe）**

- 采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。
- 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发 Concurrent Modification Exception。
- 缺点：基于拷贝内容的优点是避免了 Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。
- 场景：java.util.concurrent 包下的容器都是安全失败，可以在多线程下并发使用，并发修改，比如 CopyOnWriteArrayList 类。

11. 有哪几种实现 ArrayList 线程安全的方法?
可以使用 `Collections.synchronizedList()` 方法，它将返回一个线程安全的 List。
或者直接用 CopyOnWriteArrayList, 写时复刻

12. ArrayLsit 和 Vector 的区别
Vector 属于 JDK 1.0 时期的遗留类，已不推荐使用.
ArrayList 是在 JDK 1.2 时引入的，用于替代 Vector 作为主要的非同步动态数组实现。

### part 7
1. CopyOnWriteArrayLsit 了解多少?
CopyOnWriteArrayList 就是线程安全版本的 ArrayList。
它的名字叫`CopyOnWrite`——写时复制，已经明示了它的原理。
CopyOnWriteArrayList `采用了一种读写分离的并发策略`。CopyOnWriteArrayList 容器允许并发读，读操作是无锁的，性能较高。至于写操作，比如向容器中添加一个元素，则首先将当前容器复制一份，然后在新副本上执行写操作，结束之后再将原容器的引用指向新容器。

2. 能说一下 HashMap 的底层数据结构吗?
JDK 8 中 HashMap 的数据结构是`数组`+`链表`+`红黑树`。
![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-8.png)
HashMap 的核心是一个动态数组（`Node[] table`），用于存储键值对。这个数组的每个元素称为一个“桶”（Bucket），每个桶的索引是通过对键的哈希值进行哈希函数处理得到的。

当多个键经哈希处理后得到相同的索引时，会发生哈希冲突。HashMap 通过链表来解决哈希冲突——即将具有相同索引的键值对通过链表连接起来。

不过，链表过长时，查询效率会比较低，于是当链表的长度超过 8 时（且数组的长度大于 64），链表就会转换为红黑树。红黑树的查询效率是 O(logn)，比链表的 O(n) 要快。数组的查询效率是 O(1)。

当向 HashMap 中添加一个键值对时，会使用哈希函数计算键的哈希码，确定其在数组中的位置，`哈希函数的目标是尽量减少哈希冲突，保证元素能够均匀地分布在数组的每个位置上`

当向 HashMap 中添加元素时，如果该位置已有元素（发生哈希冲突），则新元素将被添加到链表的末尾或红黑树中。如果键已经存在，其对应的值将被新值覆盖。 

当从 HashMap 中获取元素时，也会使用哈希函数计算键的位置，然后根据位置在数组、链表或者红黑树中查找元素。

HashMap 的初始容量是 16，随着元素的不断添加，达到阈值的时候，就需要进行扩容，阈值是`capacity * loadFactor`，capacity 为容量，loadFactor 为负载因子，默认为 0.75。 

扩容后的数组大小是原来的 2 倍，`然后把原来的元素重新计算哈希值，放到新的数组中。`

总的来说，HashMap 是一种通过哈希表实现的键值对集合，它通过将键哈希化成数组索引，并在冲突时使用链表或红黑树来存储元素，从而实现快速的查找、插入和删除操作。

3. 你对红黑树了解多少？为什么不用二叉树/平衡树呢
红黑树是一种自平衡的二叉查找树:
每个节点要么是红色, 要么是黑色
根节点永远是黑色
所有的叶子节点都是黑色的
红色节点的子节点一定是黑色
从任一节点到其每个叶子的所有简单路劲都包含相同数目的黑色节点

为什么不用二叉树?
二叉树是最基本的树结构，每个节点最多有两个子节点，但是二叉树容易出现极端情况，比如`插入的数据是有序的，那么二叉树就会退化成链表`，查询效率就会变成 O(n)。

为什么不用平衡二叉树?
平衡二叉树比红黑树的要求更高，每个节点的左右子树的高度最多相差 1，这保证了极佳的查找效率，但在进行插入和删除操作时，可能需要频繁地进行旋转来维持树的平衡，这在某些情况下可能导致更高的维护成本。

红黑树是一种折中的方案，它在保证了树平衡的同时，插入和删除操作的性能也得到了保证，查询效率是 O(logn)。

#todo
4. 红黑树怎么保持平衡
红黑树有两种方式保持平衡: 旋转 和 染色
 旋转: 旋转分为两种, 左旋 和 右旋


5. HashMap 怎么查找元素呢?
![[Pasted image 20240830074133.png]]

先根据 key 的值计算哈希值, 再判断数组是否为空, 让后计算索引, 找到相应位置的第一个节点进行匹配, 匹配成功返回, 否则看看它是链表还是红黑树, 然后相应的方式进行查找

6. HashMap 的 hash 函数是怎么设计的?
HashMap 的哈希函数是先拿到 key 的 hashcode，是一个 32 位的 int 类型的数值，然后让 hashcode 的高 16 位和低 16 位进行异或操作。这么设计是为了降低哈希碰撞的概率

### part 8
1. HashMap 的 put 流程知道吗?
![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-13.jpg)

第一步, 通过 hash 方法计算 key 的哈希值
第二步, 数组进行第一次扩容
第三步, 根据哈希值计算 key 在数组中的下标，如果对应下标正好没有存放数据，则直接插入。如果对应下标已经有数据了，就需要判断是否为相同的 key，是则覆盖 value，否则需要判断是否为树节点，是则向树中插入节点，否则向链表中插入数据。
所有元素处理完后，还需要判断是否`超过`阈值`threshold`，超过则扩容。

2. 只重写 equals 没重写 hashcode, map put 的时候会发生什么
那么会导致 equals 相等的两个对象，因为 hashcode 不相等被放到不同的桶中，这样就会导致 get 的时候，找不到对应的值。

3. HashMap 怎么查找元素呢?
![[Pasted image 20240902055815.png]]
1. 使用扰动函数，获取新的哈希值
2. 计算数组下标，获取节点
3. 当前节点和 key 匹配，直接返回
4. 否则，当前节点是否为树节点，查找红黑树
5. 否则，遍历链表查找

4. HashMap 的 hash 函数是怎么设计的
HashMap 的哈希函数是先拿到 key 的 hashcode，是一个 32 位的 int 类型的数值，然后让 hashcode 的高 16 位和低 16 位进行异或操作。
这么设计是为了降低哈希碰撞的概率

#todo 
5. 为什么 hash 函数能降哈希碰撞

6. 为什么 HashMap 的容量是 2 的倍数
与其说是 2 的倍数，不如说是 2 的整数次幂, 是为了让位运算取代模运算, 降低计算机操作成本, 快速定位元素下标, 

HashMap 在定位元素位置时，再通过 `hash & (n-1)` (哈希方法的返回值 与一下 数组长度减1)来定位元素位置的，n 为数组的大小，也就是 HashMap 的容量。

因为（数组长度-1）正好相当于一个“低位掩码”——这个掩码的低位最好全是 1，这样 & 操作才有意义，否则结果就肯定是 0。

> a&b 操作的结果是：a、b 中对应位同时为 1，则对应结果位为 1，否则为 0。例如 5&3=1，5 的二进制是 0101，3 的二进制是 0011，5&3=0001=1。

2 的整次幂（或者叫 2 的整数倍）刚好是偶数，偶数-1 是奇数，奇数的二进制最后一位是 1，保证了 `hash &(length-1)` 的最后一位可能为 0，也可能为 1（取决于 hash 的值），即 & 运算后的结果可能为偶数，也可能为奇数，这样便可以保证哈希值的均匀分布。

换句话说，& 操作的结果就是将哈希值的高位全部归零，只保留低位值。

7. hashCode 对数组长度取模定位数组下标, 这块有没有优化策略


8. 如果初始化 HashMap, 传一个 17 容量, 它会怎么处理?
HashMap 会将这个值转换为大于或等于 17 的最小的 2 的幂。这是因为 HashMap 的设计是基于哈希表的，而哈希表的大小最好是 2 的幂，这样可以优化后续取模的运算，并减少哈希冲突。\\\

9. 初始化 HashMap 的时候需要传入容量值吗?
在创建 HashMap 时可以指定初始容量值。这个容量是指 Map 内部用于存储数据的数组大小。

如果预先知道 Map 将存储大量键值对，提前指定一个足够大的初始容量可以减少`因扩容导致`的重哈希（rehashing）操作，从而提高性能。

因为每次扩容时，HashMap 需要新分配一个更大的数组并重新将现有的元素插入到这个新数组中，这个过程相对耗时，尤其是当 Map 中已有大量数据时。

当然了，过大的初始容量会浪费内存，特别是当实际存储的元素远少于初始容量时。如果不指定初始容量，HashMap 将使用默认的初始容量 16。

10. 你还知道哪些哈希函数构造方法呢
HashMap 里哈希构造函数的方法叫：

- **除留取余法**：`H(key)=key%p(p<=N)`，关键字除以一个不大于哈希表长度的正整数 p，所得余数为地址，当然 HashMap 里进行了优化改造，效率更高，散列也更均衡。

除此之外，还有这几种常见的哈希函数构造方法：

- **直接定址法**
    
    直接根据`key`来映射到对应的数组位置，例如 1232 放到下标 1232 的位置。
    
- **数字分析法**
    
    取`key`的某些数字（例如十位和百位）作为映射的位置
    
- **平方取中法**
    
    取`key`平方的中间几位作为映射的位置
    
- **折叠法**
    
    将`key`分割成位数相同的几段，然后把它们的叠加和作为映射的位置

11. 解决哈希冲突的方法有哪些
解决哈希冲突的方法我了解到的有3种
①、再哈希法

准备两套哈希算法，当发生哈希冲突的时候，使用另外一种哈希算法，直到找到空槽为止。对哈希算法的设计要求比较高。

②、开放地址法

遇到哈希冲突的时候，就去寻找下一个空的槽。有 3 种方法：

- 线性探测：从冲突的位置开始，依次往后找，直到找到空槽。
- 二次探测：从冲突的位置 x 开始，第一次增加 $1^2$ 个位置，第二次增加 $2^2$，直到找到空槽。
- 双重哈希：和再哈希法类似，准备多个哈希函数，发生冲突的时候，使用另外一个哈希函数。

![三分恶面渣逆袭：拉链法 VS 开放地址法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-20.png)

三分恶面渣逆袭：拉链法 VS 开放地址法

③、拉链法

也就是所谓的链地址法，当发生哈希冲突的时候，使用链表将冲突的元素串起来。HashMap 采用的正是拉链法。

12. 怎么判断 key 相等呢
HashMap 判断两个 key 是否相等, 依赖于 key 的 equals() 方法 和 hashCode() 方法 以及 == 运算符号

在 hashcode 相等的基础上, 测试euqals() 或 == 是否返回真 true

```shell
docker run -d --name nacos --env MODE=standalone -p 8848:8848 -p 9848:9848 nacos/nacos-server:v2.2.3

docker cp nacos:/home/nacos/conf D:\docker\nacos
docker cp nacos:/home/nacos/data D:\docker\nacos
```

### part 9
1. 为什么 HashMap 链表转红黑树的阈值是8
树化发生在 table 数组的长度大于 64，且链表的长度大于 8 的时候。

为什么是 8 呢？源码的注释也给出了答案。

红黑树节点的大小大概是普通节点大小的两倍, 所以转红黑树牺牲了空间换时间, 更多的是一种兜底策略, 保证极端情况下的查找效率.
阈值为什么要选 8 呢？和统计学有关。理想情况下，使用随机哈希码，链表里的节点符合泊松分布，出现节点个数的概率是递减的，节点个数为 8 的情况，发生概率仅为`0.00000006`。

至于红黑树转回链表的阈值为什么是 6，而不是 8？是因为如果这个阈值也设置成 8，假如发生碰撞，节点增减刚好在 8 附近，会发生链表和红黑树的不断转换，导致资源浪费。

2. 扩容在什么时候扩容呢? 为什么扩容因子是0.75呢?
HashMap 会在存储的键值对数量超过阈值（即容量 * 加载因子）时进行扩容。默认的加载因子是 0.75，这意味着当 HashMap 填满了大约 75%的容量时，就会进行扩容
默认的初始容量是 16，那就是大于`16x0.75=12`时，就会触发第一次扩容操作。

3. 那么为什么选择了 0.75 作为 HashMap 的默认加载因子呢

简单来说，这是对`空间`成本和`时间`成本平衡的考虑。

在 HashMap 中有这样一段注释：
我们都知道，HashMap 的散列构造方式是 Hash 取余，负载因子决定元素个数达到多少时候扩容。

假如我们设的比较大，元素比较多，空位比较少的时候才扩容，那么发生哈希冲突的概率就增加了，查找的时间成本就增加了。

我们设的比较小的话，元素比较少，空位比较多的时候就扩容了，发生哈希碰撞的概率就降低了，查找时间成本降低，但是就需要更多的空间去存储元素，空间成本就增加了。

4. 扩容机制了解吗
扩容时, HashMap 会创建一个新的数组, 其容量是原数组的两倍, 然后将键值对放在新计算出的索引位上. 一部分索引不变, 另一部分索引为 "原索引 + 旧容量"

那你说说扩容的时候每个节点都要进行位运算吗，如果我这个 HashMap 里面有几十万条数据，都要进行位运算吗？

在 JDK 8 的新 hash 算法下，数组扩容后的索引位置，要么就是原来的索引位置，要么就是“原索引+原来的容量”，遵循一定的规律。

具体来说，就是判断原哈希值的高位中新增的那一位是否为 1，如果是，该元素会被移动到原位置加上旧容量的位置；如果不是，则保持在原位置。

所以，尽管有几十万条数据，每个数据项的位置决定仅需要一次简单的位运算。计算机操作成本还是比较低的

5. JDK 8 对 HashMap 主要做了哪些优化呢? 为什么?
相比较 JDK 7，JDK 8 的 HashMap 主要做了四点优化：
①、底层数据结构由数组 + 链表改成了数组 + 链表或红黑树的结构。
原因：如果多个键映射到了同一个哈希值，链表会变得很长，在最坏的情况下，当所有的键都映射到同一个桶中时，性能会退化到 O(n)，而红黑树的时间复杂度是 O(logn)。
②、链表的插入方式由头插法改为了尾插法。
原因：头插法虽然简单快捷，但扩容后容易改变原来链表的顺序。
③、扩容的时机由插入时判断改为插入后判断。
原因：可以避免在每次插入时都进行不必要的扩容检查，因为有可能插入后仍然不需要扩容。
④、优化了哈希算法。
JDK 7 进行了多次移位和异或操作来计算元素的哈希值。

![二哥的 Java 进阶之路：JDK 7 的 hash 方法](https://cdn.tobebetterjavaer.com/stutymore/collection-20240512093223.png)

二哥的 Java 进阶之路：JDK 7 的 hash 方法

JDK 8 优化了这个算法，只进行了一次异或操作，但仍然能有效地减少冲突。

![二哥的 Java 进阶之路：JDK 8 的 hash 方法](https://cdn.tobebetterjavaer.com/stutymore/collection-20240512093327.png)

二哥的 Java 进阶之路：JDK 8 的 hash 方法

并且能够保证扩容后，元素的新位置要么是原位置，要么是原位置加上旧容量大小。

#todo 
6. 你能自己设计实现一个 HashMap 吗?

7. HashMap 是线程安全的吗? 多线程下会有什么问题
HashMap 不是线程安全的, 主要有以下几个问题：
①、多线程下扩容会死循环。JDK1.7 中的 HashMap 使用的是头插法插入元素，在多线程的环境下，扩容的时候就有可能导致出现环形链表，造成死循环。

![二哥的 Java 进阶之路](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/collection/hashmap-thread-nosafe-07.png)

二哥的 Java 进阶之路

不过，JDK 8 时已经修复了这个问题，扩容时会保持链表原来的顺序。

②、多线程的 put 可能会导致元素的丢失。因为计算出来的位置可能会被其他线程的 put 覆盖。本来哈希冲突是应该用链表的，但多线程时由于没有加锁，相同位置的元素可能就被干掉了。

![二哥的 Java 进阶之路](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/collection/hashmap-thread-nosafe-10.png)

二哥的 Java 进阶之路

③、put 和 get 并发时，可能导致 get 为 null。线程 1 执行 put 时，因为元素个数超出阈值而导致出现扩容，线程 2 此时执行 get，就有可能出现这个问题。

![二哥的 Java 进阶之路：源码截图](https://cdn.tobebetterjavaer.com/stutymore/collection-20240326085630.png)

二哥的 Java 进阶之路：源码截图

因为线程 1 执行完 table = newTab 之后，线程 2 中的 table 此时也发生了变化，此时去 get 的时候当然会 get 到 null 了，因为元素还没有转移。


8. 有什么办法能解决 HashMap 线程不安全的问题呢？

在 Java 中，有 3 种线程安全的 Map 实现，最常用的是[ConcurrentHashMap](https://javabetter.cn/thread/ConcurrentHashMap.html)和`Collections.synchronizedMap(Map)`包装器。

Hashtable 也是线程安全的，但它的使用已经不再推荐使用，因为 ConcurrentHashMap 提供了更高的并发性和性能。

①、HashTable 是直接在方法上加 [synchronized 关键字](https://javabetter.cn/thread/synchronized-1.html)，比较粗暴。

![二哥的 Java 进阶之路：HashTable](https://cdn.tobebetterjavaer.com/stutymore/collection-20240323125211.png)

二哥的 Java 进阶之路：HashTable

②、`Collections.synchronizedMap` 返回的是 [Collections](https://javabetter.cn/common-tool/collections.html) 工具类的内部类。

![二哥的 Java 进阶之路：Collections.synchronizedMap](https://cdn.tobebetterjavaer.com/stutymore/collection-20240323125418.png)

二哥的 Java 进阶之路：Collections.synchronizedMap

内部是通过 synchronized 对象锁来保证线程安全的。

③、[ConcurrentHashMap](https://javabetter.cn/thread/ConcurrentHashMap.html) 在 JDK 7 中使用分段锁，在 JKD 8 中使用了 [CAS（Compare-And-Swap）](https://javabetter.cn/thread/cas.html)+ [synchronized 关键字](https://javabetter.cn/thread/synchronized-1.html)，性能得到进一步提升。

![初念初恋：ConcurrentHashMap 8 中的实现](https://cdn.tobebetterjavaer.com/stutymore/map-20230816155924.png)


9. HashMap 内部节点是有序的吗?
HashMap 是无序的, 根据 hash 值随机插入. 如果想使用有序的 Map, 可以使用 LinkedHashMap 或者 TreeMap。

10. 讲讲 LinkedHashMap 是怎么实现有序的?
LinkedHashMap 维护了一个双向链表，有头尾节点，同时 LinkedHashMap 节点 Entry 内部除了继承 HashMap 的 Node 属性，还有 before 和 after 用于标识前置节点和后置节点。

![Entry节点](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-33.png)

可以按照插入顺序把所有节点给串起来

![LinkedHashMap实现原理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-34.png)

LinkedHashMap实现原理

11. 讲讲 TreeMap 怎么实现有序的?
TreeMap 通过 key 的比较器来决定元素的顺序，如果没有指定比较器，那么 key 必须实现 [Comparable 接口](https://javabetter.cn/collection/comparable-omparator.html)
TreeMap 的底层是红黑树，红黑树是一种自平衡的二叉查找树，每个节点都大于其左子树中的任何节点，小于其右子节点树种的任何节点。
插入或者删除元素时通过旋转和着色来保持树的平衡。
查找的时候通过从根节点开始，利用二叉查找树的性质，逐步向左或者右子树递归查找，直到找到目标元素。

### part 10
1. TreeMap 和 HashMap 的区别
①、HashMap 是基于数组+链表+红黑树实现的，put 元素的时候会先计算 key 的哈希值，然后通过哈希值计算出数组的索引，然后将元素插入到数组中，如果发生哈希冲突，会使用链表来解决，如果链表长度大于 8，会转换为红黑树。

get 元素的时候同样会先计算 key 的哈希值，然后通过哈希值计算出数组的索引，如果遇到链表或者红黑树，会通过 key 的 equals 方法来判断是否是要找的元素。

②、TreeMap 是基于红黑树实现的，put 元素的时候会先判断根节点是否为空，如果为空，直接插入到根节点，如果不为空，会通过 key 的比较器来判断元素应该插入到左子树还是右子树。

get 元素的时候会通过 key 的比较器来判断元素的位置，然后递归查找。

由于 HashMap 是基于哈希表实现的，所以在没有发生哈希冲突的情况下，HashMap 的查找效率是 O(1)。适用于查找操作比较频繁的场景。

而 TreeMap 是基于红黑树实现的，所以 TreeMap 的查找效率是 O(logn)。并且保证了元素的顺序，因此适用于需要大量范围查找或者有序遍历的场景。

2. 讲讲 HashSet 的底层实现
HashSet 其实是由 HashMap 实现的，只不过值由一个固定的 Object 对象填充，而键用于操作。

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    static final long serialVersionUID = -5024744406713321676L;
    private transient HashMap<E,Object> map;
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    // ……
}
```

实际开发中，HashSet 并不常用，比如，如果我们需要按照顺序存储一组元素，那么 ArrayList 和 LinkedList 可能更适合；如果我们需要存储键值对并根据键进行查找，那么 HashMap 可能更适合。

HashSet 主要用于去重，比如，我们需要统计一篇文章中有多少个不重复的单词，就可以使用 HashSet 来实现。

```java
// 创建一个 HashSet 对象
HashSet<String> set = new HashSet<>();

// 添加元素
set.add("沉默");
set.add("王二");
set.add("陈清扬");
set.add("沉默");

// 输出 HashSet 的元素个数
System.out.println("HashSet size: " + set.size()); // output: 3

// 遍历 HashSet
for (String s : set) {
    System.out.println(s);
}
```

HashSet 会自动去重，因为它是用 HashMap 实现的，HashMap 的键是唯一的（哈希值），相同键的值会覆盖掉原来的值，于是第二次 set.add("沉默") 的时候就覆盖了第一次的 set.add("沉默")。




![三分恶面渣逆袭：HashSet套娃](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-36.png)

三分恶面渣逆袭：HashSet套娃


HashSet 与 ArrayList 的区别
- ArrayList 是基于动态数组实现的，HashSet 是基于 HashMap 实现的。
- ArrayList 允许重复元素和 null 值，可以有多个相同的元素；HashSet 保证每个元素唯一，不允许重复元素，基于元素的 hashCode 和 equals 方法来确定元素的唯一性。
- ArrayList 保持元素的插入顺序，可以通过索引访问元素；HashSet 不保证元素的顺序，元素的存储顺序依赖于哈希算法，并且可能随着元素的添加或删除而改变。

HashSet 怎么判断元素重复, 重复了是否put
HashSet 的 add 方法是通过调用 HashMap 的 put 方法实现的：

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

所以 HashSet 判断元素重复的逻辑底层依然是 HashMap 的底层逻辑：