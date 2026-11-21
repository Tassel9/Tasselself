# Java集合

[返回模块目录](../README.md) · [打开完整总文档](../Java后端八股.md)

## 高频题

### 集合

#### Java中的线程安全的集合是什么？

常见线程安全集合有：

1. 老的同步集合：`Vector` 、`Hashtable` 。

2. `Collections`  工具类包装出来的同步集合：`Collections.synchronizedList()` 、`Collections.synchronizedMap()` 、`Collections.synchronizedSet()` 。

3. JUC 包下的并发集合：`ConcurrentHashMap` 、`CopyOnWriteArrayList` 、`CopyOnWriteArraySet` 、`ConcurrentLinkedQueue` 、`BlockingQueue`  系列。

面试中重点回答 `ConcurrentHashMap`  和 `CopyOnWriteArrayList` 。

#### 集合的分类有哪些？

Java 集合主要分为两类：`Collection`  单列集合和 `Map`  双列集合。`Collection`  下面常见的有：

1. `List` ：有序、可重复，常见实现类有 `ArrayList` 、`LinkedList` 。

2. `Set` ：不允许重复，常见实现类有 `HashSet` 、`LinkedHashSet` 、`TreeSet` 。

3. `Queue` ：队列结构，常见实现类有 `ArrayDeque` 、`PriorityQueue` ，并发场景下还有各种`BlockingQueue` 。

`Map`  是双列集合，用来存储键值对，常见实现类有 `HashMap` 、`LinkedHashMap` 、`TreeMap` 、`Hashtable` 、`ConcurrentHashMap` 。

简单来说，`List`  关注顺序和重复，`Set`  关注去重，`Queue`  关注队列语义，`Map`  关注通过 key快速找到 value。

### List

#### arraylist和linkedlist区别

1. 是否保证线程安全： `ArrayList`  和 `LinkedList`  都不是线程安全的。

2. 底层数据结构： `ArrayList`  底层是 `Object[]`  数组；`LinkedList`  底层是双向链表。

3. 随机访问： `ArrayList`  支持快速随机访问，因为可以直接通过下标访问元素，时间复杂度是O(1)。`LinkedList`  不支持高效随机访问，需要从头或尾开始遍历，时间复杂度是 O(n)。

4. 插入和删除：

    - `ArrayList`  在尾部添加元素通常是 O(1)，但是如果触发扩容，需要复制数组，成本就是O(n)。如果在指定位置插入或删除元素，需要移动后面的元素，时间复杂度也是 O(n)。

    - `LinkedList`  在头尾插入或删除元素是 O(1)，但是如果是在指定位置插入或删除，也需要先遍历到指定位置，所以整体仍然是 O(n)。

5. 内存占用： `ArrayList`  主要是数组可能预留一部分空间；`LinkedList`  每个节点除了存储数据，还要存储前驱和后继指针，所以单个元素占用空间更多。

总结：如果查询多、按下标访问多，一般用 `ArrayList` 。如果主要是在头尾增删，可以考虑`LinkedList` ，但是实际开发中 `ArrayList`  更常用。

#### 为什么 new ArrayList<>（）时建议指定初始化容量值？

因为 `ArrayList`  底层是数组，如果容量不够就会扩容。

扩容时会创建一个新数组，然后把旧数组中的元素复制到新数组中。这个过程会带来性能损耗。

如果一开始能预估集合大小，指定一个合适的初始化容量，就可以尽量减少扩容次数，从而提高性能，也能避免不必要的内存浪费。

例如：

```java
List<User> users = new ArrayList<>(1000);
```

需要注意的是，`new ArrayList<>(1000)`  只是指定容量，不代表集合中已经有 1000 个元素，此时 `size`  仍然是 0。

#### 为什么 ArrayList 默认情况下的扩容机制是扩容为原数组的1.5倍？

这是内存使用和性能之间的权衡。

如果扩容因子太大，可能申请了很多用不到的空间，造成内存浪费。

如果扩容因子太小，就会频繁扩容，每次扩容都要复制数组，造成性能浪费。

所以扩容为原数组的 1.5 倍，是一个比较折中的经验值，适合大多数场景。

#### ArrayList是线程的安全吗？

`ArrayList`  不是线程安全的。

如果 `ArrayList`  作为多线程共享数据，比如单例对象中的成员变量、静态变量等，那么多个线程并发修改时可能会出现线程安全问题。

例如：

1. 多个线程同时 `add()` ，可能导致元素覆盖。

2. 多个线程同时扩容，可能导致数据丢失。

解决方案：

1. 使用 `synchronized`  包裹对集合的操作。

2. 使用 `Collections.synchronizedList(new ArrayList<>())` 。

3. 使用 `CopyOnWriteArrayList` 。

#### CopyOnWriteArrayList 的实现原理

`CopyOnWriteArrayList`  底层也是数组。

它的核心思想是写时复制：

1. 读操作直接读取原数组，不加锁。

2. 写操作会先加锁。

3. 写入时不会直接修改原数组，而是复制一个新数组。

4. 在新数组上完成添加、删除、修改操作。

5. 操作完成后，再把数组引用指向新数组。

优点：

`CopyOnWriteArrayList`  读操作不加锁，写操作也不会阻塞读操作，所以读性能比较好，适合读多写少的场景。

缺点：

每次写操作都要复制数组，占用更多内存，而且读取到的数据可能不是最新的，所以不适合写多或者实时性要求很高的场景。

#### list和set的区别

`List`  和 `Set`  都属于 `Collection`  单列集合。

区别：

1. `List`  有序，可以重复；`Set`  不允许重复。

2. `List`  可以通过下标访问元素；普通 `Set`  不能通过下标访问。

3. `List`  常见实现类有 `ArrayList` 、`LinkedList` ；`Set`  常见实现类有 `HashSet` 、`LinkedHashSet` 、`TreeSet` 。

4. `HashSet`  通常无序，`LinkedHashSet`  可以保持插入顺序，`TreeSet`  可以按照规则排序。

使用场景：

如果需要按顺序保存并且允许重复，用 `List` 。如果需要去重，用 `Set` 。

#### 如果要进行大量的增加和删除使用arraylist好还是linkedlist好？

不能简单地说大量增加和删除就一定用 `LinkedList` 。

如果是在头尾进行增删，`LinkedList`  不需要移动元素，效率比较高。

如果是在指定位置增删，`LinkedList`  也需要先遍历到指定位置，这一步是 O(n)，所以不一定比`ArrayList`  快。

`ArrayList`  指定位置增删需要移动元素，但是它的数组结构缓存友好，实际开发中很多场景性能反而不差。

所以一般回答：头尾增删可以考虑 `LinkedList` ，普通业务列表优先考虑 `ArrayList` 。

### Map

#### hashset和hashmap的区别

`HashSet`  底层就是基于 `HashMap`  实现的。

`HashSet`  添加元素时，本质上是把元素作为 `HashMap`  的 key 存进去，value 使用一个固定的占位对象。

所以 `HashSet`  不允许元素重复，本质原因是 `HashMap`  的 key 不允许重复。

`HashSet`  判断元素是否重复时，依赖元素的 `hashCode()`  和 `equals()`  方法。

#### equals 和 hashCode 为什么要一起重写？

在 `HashMap` 、`HashSet`  这类哈希结构中，会先通过 `hashCode()`  计算元素应该落在哪个桶里，然后再通过 `equals()`  判断元素是否真的相等。

规则：

1. 如果两个对象 `equals()`  返回 `true` ，那么这两个对象的 `hashCode()`  必须相等。

2. 如果两个对象的 `hashCode()`  相等，这两个对象不一定相等，因为可能发生哈希冲突。

如果只重写 `equals()` ，没有重写 `hashCode()` ，可能会导致两个逻辑上相等的对象落到不同桶中，最终导致 `HashMap`  查找失败，或者 `HashSet`  去重失败。

#### HashMap 底层数据结构是什么？

JDK1.7 中 `HashMap`  的底层结构是数组 + 链表。

JDK1.8 中 `HashMap`  的底层结构是数组 + 链表 + 红黑树。

数组中的每一个位置可以理解为一个桶。当多个 key 经过 hash 计算后落到同一个桶中，就产生了哈希冲突。

JDK1.8 之前主要通过链表解决冲突，JDK1.8 以后，如果链表过长，会把链表转成红黑树，提高查询效率。

#### HashMap的添加元素流程

调用 `map.put(key, value)`  时，大致流程如下：

1. 先计算 key 的 hash 值。

2. 通过 `(`数组长度 `- 1) & hash`  计算元素应该存放的数组下标。

3. 如果该位置为 `null` ，直接创建节点放进去。

4. 如果该位置不为 `null` ，说明发生哈希冲突，需要遍历链表或者红黑树。

5. 遍历过程中，如果发现 key 相同，就覆盖原来的 value。

6. 如果没有找到相同 key，就把新节点插入链表或红黑树中。JDK1.7 是头插法，JDK1.8 是尾插法。

7. JDK1.8 以后，如果链表长度大于等于 8，并且数组长度大于等于 64，会把链表转成红黑树。

8. 插入完成后，如果元素数量超过扩容阈值，就进行扩容。

#### hashmap为什么容量是2的幂次方

因为 `HashMap`  计算数组下标时使用的是：

```text
(n - 1) & hash
```

当数组长度 `n`  是 2 的幂时，`n - 1`  的二进制低位全是 1。

这样 `(n - 1) & hash`  的效果就和 `hash % n`  类似，但是位运算效率更高。

同时，数组长度是 2 的幂时，可以让 hash 分布更加均匀，减少某些位置永远无法命中的情况，从而降低哈希冲突的概率。

#### HashMap扩容加载因子为什么是0.75？

`HashMap`  默认负载因子是 `0.75` 。

也就是说，当元素数量达到数组长度的 `0.75`  倍时，就会触发扩容。

为什么是 0.75？

1. 如果负载因子太小，扩容会更频繁，空间利用率低，浪费内存。

2. 如果负载因子太大，虽然空间利用率高，但是哈希冲突会增多，查询性能会下降。

3. `0.75`  是在时间和空间之间取得的一个折中值，可以在减少哈希冲突和控制内存占用之间达到比较好的平衡。

#### hashmap扩容机制

`HashMap`  默认初始容量是 16，默认负载因子是 0.75，所以默认第一次扩容阈值是 12。

当 `HashMap`  中的元素数量超过阈值时，就会进行扩容。

扩容时，新数组容量会变成原来的 2 倍。

JDK1.8 中扩容时，元素的新位置只有两种情况：

1. 仍然在原来的位置。

2. 移动到 原位置 `+` 旧数组容量 的位置。

这样做的原因是数组长度每次都扩容为 2 倍，可以通过 hash 的某一位来判断元素扩容后的新位置，不需要重新完整取模。

#### HashMap为什么是线程不安全的

`HashMap`  不是线程安全的。

如果 `HashMap`  作为多线程共享数据，多个线程并发修改时可能会出现问题。

常见问题：

1. JDK1.7 中，多线程扩容时可能造成环形链表或数据丢失。

2. JDK1.8 中，多个线程同时往同一个空桶插入元素，可能出现数据覆盖。

3. 并发 put、resize 时，也可能出现数据丢失、读到异常数据等问题。

解决方案：

1. 使用 `synchronized`  包裹操作。

2. 使用 `Collections.synchronizedMap()` 。

3. 使用 `ConcurrentHashMap` 。

#### Hashtable与HashMap的区别

`Hashtable`  是实现 `Map`  接口的双列集合，底层基于数组 + 链表实现，key 和 value 都不允许为`null` ，并且它是线程安全的，主要通过 `synchronized`  实现。

区别：

1. 线程安全： `HashMap`  线程不安全，`Hashtable`  线程安全。

2. 效率： `HashMap`  效率更高，`Hashtable`  因为方法上大多加了 `synchronized` ，效率较低。

3. null 值： `HashMap`  允许一个 `null`  key 和多个 `null`  value；`Hashtable`  的 key 和 value都不能为 `null` 。

4. 初始容量： `HashMap`  默认初始容量是 16；`Hashtable`  默认初始容量是 11。

5. 扩容方式： `HashMap`  每次扩容为原来的 2 倍；`Hashtable`  每次扩容为 `2n + 1` 。

6. 底层结构： JDK1.8 以后 `HashMap`  是数组 + 链表 + 红黑树；`Hashtable`  没有红黑树机制。

现在 `Hashtable`  基本已经被淘汰，如果需要线程安全的 Map，一般使用`ConcurrentHashMap` 。

### ConcurrentHashMap

#### ConcurrentHashMap是悲观锁还是乐观锁?

既有乐观锁，也有悲观锁。

在 JDK1.8 的 `ConcurrentHashMap`  中：

1. 使用 CAS 初始化数组或者插入空桶头节点，这属于乐观锁思想。

2. 当桶不为空时，会使用 `synchronized`  锁住桶的头节点，再对链表或红黑树进行修改，这属于悲观锁。

为什么不全部用 CAS？

因为遍历链表、比较 key、插入节点、判断是否树化这些操作耗时更长，如果一直用 CAS 自旋，会浪费 CPU。

为什么不全部用 `synchronized` ？

因为初始化数组、空桶插入这类操作很短，用 CAS 成本更低。

所以 `ConcurrentHashMap`  是根据不同操作选择不同的并发控制方式。

#### currentHashMap如何保证线程安全的？

`ConcurrentHashMap`  是 JUC 包下的线程安全 Map。

JDK1.7 和 JDK1.8 的实现差别很大。

##### JDK1.7中的ConcurrentHashMap

JDK1.7 中 `ConcurrentHashMap`  使用的是分段锁。

底层结构是 `Segment`  数组 + `HashEntry`  数组 + 链表。

`Segment`  继承了 `ReentrantLock` ，每个 `Segment`  就是一把锁。不同线程如果操作的是不同的 `Segment` ，就可以并发执行，从而降低锁竞争。

简单来说，`Hashtable`  是锁整张表，而 JDK1.7 的 `ConcurrentHashMap`  是把表分成多个段，每个段单独加锁。

##### JDK1.8中的ConcurrentHashMap

JDK1.8 中 `ConcurrentHashMap`  的底层结构变成了 `Node`  数组 + 链表 + 红黑树。

它不再使用分段锁作为核心实现，而是使用 CAS + `synchronized` 。

put 流程大致如下：

1. 判断 key 和 value 是否为 `null` ，如果是就抛异常。

2. 如果数组还没有初始化，就使用 CAS 初始化数组。

3. 根据 key 的 hash 值定位桶位置。

4. 如果桶为空，就使用 CAS 插入头节点。

5. 如果桶不为空，并且当前位置有其他线程正在扩容，当前线程会参与协助扩容。

6. 如果桶不为空且没有在扩容，就使用 `synchronized`  锁住桶的头节点，然后遍历链表或红黑树。

7. 如果找到相同 key，就替换 value；如果没有找到，就新增节点。

8. 新增后判断是否需要树化或者扩容。

JDK1.8 的锁粒度更细，是锁住桶的头节点，而不是锁住整张表或整个段，所以并发性能更好。

#### ConcurrentHashMap为什么key和value不能为null？

因为会产生二义性问题。

如果 `get(key)`  返回 `null` ，可能表示两种情况：

1. 这个 key 不存在。

2. 这个 key 存在，但是对应的 value 就是 `null` 。

在 `HashMap`  中，可以再调用 `containsKey()`  判断。

但是在 `ConcurrentHashMap`  中，`get()`  和 `containsKey()`  之间可能有其他线程修改数据，所以不能保证判断结果准确。

为了避免这种二义性，`ConcurrentHashMap`  不允许 key 和 value 为 `null` 。

### 其他

#### 迭代器的实现原理是什么？

集合的遍历方式有迭代器遍历、增强 for 遍历、Lambda 表达式遍历等。

增强 for 遍历集合时，底层也是调用迭代器。

迭代器的好处是提供了一种统一的方式来遍历不同类型的集合，而不需要暴露底层数据结构。

如果没有迭代器，不同的数据结构就需要使用不同的遍历方式。有了迭代器之后，只需要掌握统一的`hasNext()` 、`next()` 、`remove()`  方法。

核心接口：

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    void remove();
}
```

不同集合的迭代器实现不同：

1. `ArrayList`  通过数组下标实现迭代。

2. `LinkedList`  通过双向链表指针实现迭代。

3. `HashMap`  会按照桶数组和链表/红黑树结构进行遍历。

#### fail-fast是什么？

`fail-fast`  是集合遍历时的快速失败机制。

以 `ArrayList`  为例，集合内部有一个 `modCount` ，迭代器中有一个 `expectedModCount` 。

如果在迭代过程中，使用集合自己的 `add()` 、`remove()`  等方法修改了集合结构，就可能导致`modCount`  和 `expectedModCount`  不一致，然后抛出`ConcurrentModificationException` 。

正确删除方式是使用迭代器自己的 `remove()` ：

```java
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    if ("a".equals(item)) {
        iterator.remove();
    }
}
```

或者使用 `removeIf()` 。

#### 泛型是什么？使用泛型的好处？

Java 泛型是 JDK5 引入的特性，允许在定义类、接口或方法时使用类型参数，使用时再传入具体类型。

使用泛型的好处：

1. 类型安全：编译期就能检查类型，减少运行时 `ClassCastException` 。

2. 减少强制类型转换：从集合中取出元素时，不需要手动强转。

3. 提高代码复用性：同一套代码可以适配多种类型。

例如：

```java
List<String> list = new ArrayList<>();
list.add("abc");
String value = list.get(0);
```

## 扩展题

### List

#### ArrayList扩容源码

`ArrayList`  底层是数组。

无参构造创建出来时，底层通常是一个空数组。第一次添加元素时，会扩容到默认容量 10。

后续扩容时，一般会扩容为原容量的 1.5 倍。

扩容公式可以简单理解为：

```text
newCapacity = oldCapacity + (oldCapacity >> 1);
```

扩容底层会调用类似 `Arrays.copyOf()`  的逻辑，创建新数组并复制旧数组元素。

所以如果能提前预估元素数量，最好在创建 `ArrayList`  时指定初始化容量，减少扩容次数。

#### ArrayList删除元素有哪些坑？

不要在增强 for 中直接调用集合自己的 `remove()`  方法。

错误示例：

```java
for (String item : list) {
    if ("a".equals(item)) {
        list.remove(item);
    }
}
```

因为增强 for 底层是迭代器，遍历过程中直接修改集合结构，可能会触发 `fail-fast`  机制，抛出`ConcurrentModificationException` 。

正确方式：

```java
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    if ("a".equals(iterator.next())) {
        iterator.remove();
    }
}
```

也可以使用：

```text
list.removeIf(item -> "a".equals(item));
```

#### LinkedList为什么很多场景不推荐？

`LinkedList`  是双向链表，理论上增删节点只需要修改指针。

但是实际场景中，如果是指定位置增删，需要先遍历到指定位置，这一步是 O(n)。

而且 `LinkedList`  每个节点都要额外保存前驱和后继指针，占用内存更多。

链表节点在内存中也不是连续存储，缓存局部性不如数组。

所以普通业务场景中，`ArrayList`  往往比 `LinkedList`  更常用。

如果是栈或队列场景，很多时候可以优先考虑 `ArrayDeque` 。

### Map

#### HashMap的hash扰动函数有什么用？

`HashMap`  不会直接拿 key 的 `hashCode()`  去计算下标，而是会对 hash 值做扰动。

JDK1.8 中大致是：

```text
h ^ (h >>> 16)
```

原因是 `HashMap`  通过 `(n - 1) & hash`  计算桶下标。

数组长度通常不会特别大，所以真正参与下标计算的主要是 hash 的低位。

如果只使用低位，可能会导致冲突比较严重。

扰动函数可以让高位信息也参与到低位计算中，从而让元素分布更均匀，减少哈希冲突。

#### HashMap为什么链表长度到8才树化？

链表转红黑树是为了提高查询效率。

链表查询的时间复杂度是 O(n)，红黑树查询的时间复杂度是 O(log n)。

但是红黑树维护起来比链表复杂，也会占用更多空间。

所以节点数量比较少时，使用链表反而更合适。

当链表长度达到 8 时，说明哈希冲突比较严重，这时才考虑转成红黑树。

并且不是链表长度达到 8 就一定树化，还要求数组长度至少达到 64。

如果数组长度小于 64，说明当前更可能是数组太小导致冲突多，这时优先扩容，而不是树化。

#### HashMap为什么红黑树节点小于6会退化成链表？

因为节点数量比较少时，链表遍历成本并不高。

红黑树需要维护颜色、父子节点、旋转等逻辑，结构更复杂。

所以当红黑树节点数量变少时，会退化回链表，减少维护红黑树的成本。

树化阈值是 8，退化阈值是 6，中间留了一个缓冲区，避免节点数量在 7、8 附近反复树化和退化。

#### HashMap JDK1.7和JDK1.8有什么区别？

1. 底层结构不同：

    - JDK1.7 是数组 + 链表。

    - JDK1.8 是数组 + 链表 + 红黑树。

2. 插入方式不同：

    - JDK1.7 链表使用头插法。

    - JDK1.8 链表使用尾插法。

3. 并发问题表现不同：

    - JDK1.7 在并发扩容时，头插法可能导致链表成环。

    - JDK1.8 仍然不是线程安全的，并发 put 时可能出现数据覆盖。

4. 冲突严重时处理不同：

    - JDK1.7 链表过长时查询会退化为 O(n)。

    - JDK1.8 链表过长且数组长度达到要求时，会转成红黑树，查询效率更高。

#### HashMap扩容后元素位置怎么变化？

JDK1.8 中，`HashMap`  扩容后容量变成原来的 2 倍。

因为容量是 2 的幂，所以每个元素扩容后的新位置只有两种：

1. 位置不变。

2. 位置变成 原下标 `+` 旧容量。

判断方式可以理解为：

```text
(hash & oldCapacity) == 0
```

如果结果为 0，就还在原来的位置。

如果结果不为 0，就移动到 原下标 `+` 旧容量。

这也是为什么 `HashMap`  要保持容量为 2 的幂。

#### HashMap可以用可变对象做key吗？

不推荐。

因为 `HashMap`  定位 key 时依赖 `hashCode()`  和 `equals()` 。

如果一个对象作为 key 放进 `HashMap`  之后，又修改了参与 `hashCode()`  或 `equals()`  计算的字段，那么这个对象的 hash 值可能发生变化。

此时再用这个对象去查询，可能会定位到错误的桶，导致查不到数据。

所以作为 `HashMap`  key 的对象，最好是不可变对象，例如 `String` 、包装类型，或者字段不可变的对象。

#### HashMap的时间复杂度是多少？

正常情况下，`HashMap`  的 `put()`  和 `get()`  平均时间复杂度都是 O(1)。

如果哈希冲突严重：

1. JDK1.7 中链表很长时，查询会退化成 O(n)。

2. JDK1.8 中链表转成红黑树后，查询复杂度可以降到 O(log n)。

但是这只是平均意义上的复杂度，实际性能还和 hash 分布、初始容量、负载因子、扩容次数有关。

#### LinkedHashMap有什么用？

`LinkedHashMap`  继承自 `HashMap` 。

它在 `HashMap`  的基础上维护了一条双向链表。

所以 `LinkedHashMap`  既可以像 `HashMap`  一样快速查询，又可以维护顺序。

它可以维护两种顺序：

1. 插入顺序。

2. 访问顺序。

如果开启访问顺序，可以用来实现简单的 LRU 缓存。

示例：

```java
Map<String, String> lru = new LinkedHashMap<>(16, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry<String, String> eldest) {
        return size() > 100;
    }
};
```

#### TreeMap和TreeSet底层是什么？

`TreeMap`  底层是红黑树。

它会根据 key 的自然顺序或者传入的 `Comparator`  对 key 进行排序。

`TreeSet`  底层是基于 `TreeMap`  实现的，元素作为 `TreeMap`  的 key，value 是一个固定占位对象。

特点：

1. 元素有序。

2. 增删查的时间复杂度通常是 O(log n)。

3. 元素需要具备比较能力，也就是实现 `Comparable` ，或者创建集合时传入 `Comparator` 。

适用场景：

需要排序、范围查询时，可以使用 `TreeMap`  或 `TreeSet` 。

### ConcurrentHashMap

#### ConcurrentHashMap JDK1.8 put流程细节

JDK1.8 中 `ConcurrentHashMap`  的 put 流程：

1. 判断 key 和 value 是否为 `null` ，如果是就抛异常。

2. 如果数组还没有初始化，就使用 CAS 初始化数组。

3. 根据 hash 值计算桶位置。

4. 如果桶位置为空，就使用 CAS 插入头节点。

5. 如果桶位置不为空，先判断当前桶是否正在扩容，如果正在扩容，当前线程会协助扩容。

6. 如果没有在扩容，就使用 `synchronized`  锁住桶的头节点。

7. 如果桶中是链表，就遍历链表，key 相同则替换 value，key 不同则新增节点。

8. 如果桶中是红黑树，就按照红黑树规则进行插入或替换。

9. 新增节点后判断是否需要树化。

10. 最后更新元素数量，并判断是否需要扩容。

#### ConcurrentHashMap get为什么通常不加锁？

`ConcurrentHashMap`  的 `get()`  通常不需要加锁。

原因是 `get()`  操作不修改集合结构，只是根据 hash 定位桶，然后遍历链表或红黑树查找数据。

同时，`ConcurrentHashMap`  内部通过 `volatile`  等机制保证关键字段的可见性。

所以读操作可以做到高并发。

需要注意的是，它并不是提供整张表的强一致快照，而是保证单次操作在并发环境下的线程安全和可见性。

#### ConcurrentHashMap扩容为什么可以多线程协助？

扩容迁移数据的成本比较高。

如果只有一个线程负责扩容，其他线程都等待，效率会比较低。

JDK1.8 中 `ConcurrentHashMap`  支持多个线程协助扩容。

当线程发现当前桶正在扩容时，可以参与数据迁移，把不同区间的数据迁移到新数组中。

这样可以提高扩容效率，减少单个线程扩容带来的阻塞时间。

#### ConcurrentHashMap为什么比Hashtable快？

`Hashtable`  主要是通过 `synchronized`  锁住整个方法来保证线程安全，锁粒度比较粗。

也就是说，多线程访问时，很容易出现大量锁竞争。

`ConcurrentHashMap`  的锁粒度更细：

1. JDK1.7 中使用分段锁，不同段可以并发操作。

2. JDK1.8 中使用 CAS + `synchronized` ，锁住的是桶的头节点。

3. 读操作通常不加锁。

所以 `ConcurrentHashMap`  的并发性能比 `Hashtable`  好很多。

#### Collections.synchronizedMap和ConcurrentHashMap区别

`Collections.synchronizedMap()`  是对普通 Map 做同步包装。

它的实现方式比较简单，很多操作都是通过同一把锁来保证线程安全，所以锁粒度比较粗。

`ConcurrentHashMap`  是专门为并发场景设计的 Map。

区别：

1. `synchronizedMap`  通常锁整个 Map；`ConcurrentHashMap`  锁粒度更细。

2. `synchronizedMap`  并发性能较低；`ConcurrentHashMap`  并发性能更好。

3. `synchronizedMap`  迭代时通常还需要手动加锁；`ConcurrentHashMap`  的迭代器是弱一致的。

4. 如果底层是 `HashMap` ，`synchronizedMap`  可以允许 `null`  key/value；`ConcurrentHashMap`  不允许 key 和 value 为 `null` 。

### queue

#### PriorityQueue底层是什么？

`PriorityQueue`  底层是堆，默认是小顶堆。

它不是普通的先进先出队列。

每次调用 `poll()` ，取出的是优先级最高的元素，默认情况下也就是最小的元素。

特点：

1. 获取堆顶元素是 O(1)。

2. 插入元素和删除堆顶元素通常是 O(log n)。

3. 不是线程安全的。

如果并发场景需要优先级阻塞队列，可以使用 `PriorityBlockingQueue` 。

#### BlockingQueue常见实现有哪些？

`BlockingQueue`  是阻塞队列，常用于生产者消费者模型和线程池。

常见实现：

1. `ArrayBlockingQueue` ：基于数组的有界阻塞队列，FIFO。

2. `LinkedBlockingQueue` ：基于链表的阻塞队列，可以指定容量，如果不指定容量，容量接近`Integer.MAX_VALUE` 。

3. `SynchronousQueue` ：不存储元素，生产者放入任务必须等待消费者接收。

4. `PriorityBlockingQueue` ：支持优先级的无界阻塞队列。

5. `DelayQueue` ：延迟队列，元素到期后才能被取出。

线程池中经常会问工作队列：

`ArrayBlockingQueue`  有界，可以避免任务无限堆积。

`LinkedBlockingQueue`  如果不指定容量，可能造成任务大量堆积，带来内存风险。

`SynchronousQueue`  不缓存任务，适合任务直接移交。

### 其他

#### fail-fast和fail-safe有什么区别？

`fail-fast`  是快速失败。

在遍历集合时，如果发现集合结构被非迭代器自身的方法修改，就会抛出`ConcurrentModificationException` 。

常见于 `ArrayList` 、`HashMap`  这类非并发集合。

`fail-safe`  可以理解为安全失败。

它在遍历时不会直接因为集合被修改就抛异常。

例如：

1. `CopyOnWriteArrayList`  遍历的是数组快照。

2. `ConcurrentHashMap`  的迭代器是弱一致的。

需要注意的是，`ConcurrentModificationException`  不能用来做并发控制，它只是集合发现异常修改时的一种检测机制。

#### Iterator、Iterable、增强for的关系

`Iterable`  表示一个对象可以被迭代，核心方法是 `iterator()` 。

`Iterator`  表示具体的迭代器，核心方法是 `hasNext()` 、`next()` 、`remove()` 。

增强 for 遍历集合时，底层就是调用 `Iterable`  的 `iterator()`  方法拿到迭代器，然后通过迭代器遍历。

所以只要一个类实现了 `Iterable`  接口，就可以使用增强 for 遍历。

#### 泛型类型擦除会带来哪些影响？

Java 泛型在编译期做类型检查，运行期大部分泛型信息会被擦除。

影响：

1. 不能直接 `new T()` 。

2. 不能直接创建泛型数组，例如 `new List<String>[10]` 。

3. `List<String>`  和 `List<Integer>`  在运行期都是 `List` 。

4. 泛型不能直接使用基本类型，只能使用包装类型，例如 `List<Integer>` 。

类型擦除的根本原因是兼容性。

Java 在 JDK5 才引入泛型，为了兼容之前没有泛型的代码和 JVM，所以采用了类型擦除的方式。

#### TreeSet去重和HashSet去重有什么区别？

`HashSet`  去重依赖 `hashCode()`  和 `equals()` 。

`TreeSet`  去重依赖比较规则。

如果使用自然排序，就依赖元素的 `compareTo()` 。

如果传入了比较器，就依赖 `Comparator`  的 `compare()` 。

当比较结果为 0 时，`TreeSet`  会认为两个元素相同。

所以使用 `TreeSet`  时，需要保证比较规则和业务上的相等规则一致，否则可能出现元素被错误去重的问题。

### 面试中容易混淆的点

1. `LinkedList`  不是所有增删都快，指定位置增删仍然要先遍历。

2. `HashMap`  链表长度达到 8 不一定树化，还要看数组长度是否达到 64。

3. JDK1.8 的 `ConcurrentHashMap`  不是继续使用分段锁作为核心实现，而是 CAS +`synchronized` 。

4. `ConcurrentHashMap`  不能存 `null`  key 和 `null`  value。

5. `CopyOnWriteArrayList`  只适合读多写少，不适合写多场景。

6. `Collections.synchronizedXXX`  的复合操作和迭代操作通常还需要额外加锁。

7. `HashSet`  去重失败，通常是 `equals()`  和 `hashCode()`  没有一起正确重写。

8. `TreeSet`  是否重复，看的是比较结果是不是 0，不只是看 `equals()` 。

9. 增强 for 底层是迭代器，遍历时不要直接用集合自己的 `remove()` 。

10. 可变对象不适合作为 `HashMap`  的 key。

## 后端面经与场景补充

<!-- 原PDF第478页；面经原题号 11 -->

### java当中的集合说几个并且要说一下底层

原文未附答案。
