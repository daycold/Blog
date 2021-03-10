# COLLECTION
## stream methods
### kotlin
#### Collection
Collection 实现了 Iterable， 持有一个 iterator(): Iterator 方法
list 的 filter 实际调用的是 Iterable.filterTo(ArrayList(), perdicate: (T) -> Boolean), 返回一个 MutableCollection。
每次链式调用都是使用新生成的 Iterable 进行调用。每次调用都会遍历一遍 iterator
#### Sequence
接口申明一个方法 iterator(): Iterator
调用 foreach，map， filter 等方法会新建一个 Sequence 实例

    public fun <T> Sequence<T>.filter(predicate: (T) -> Boolean): Sequence<T> {
        return FilteringSequence(this, true, predicate)
    }

每个 sequence 持有前一个 sequence 的引用，公用同一个 iterator，链式调用会发生在 iterator 的方法的实现上。只会遍历一次 iterator

    override fun iterator(): Iterator<T> = object : Iterator<T> {
        val iterator = sequence.iterator()
        ......
    }

    val list = (0..100000).toList()
    val mills1 = System.currentTimeMillis()
    list.asSequence().filter { it > 300 }.map { it.toString() }
    println(System.currentTimeMillis() - mills1)
    输出结果为 29,加toMutableList()时为 76,加 toList()时为 78
    val mills2 = System.currentTimeMillis()
    val list2 = list.filter { it > 300 }.map { it.toString() }
    println(System.currentTimeMillis() - mills2)
    输出结果为34
    List<Integer> list = new ArrayList<>(100001);
    for (int i = 0; i <= 100000; i++) {
        list.add(i);
    }
    long mills1 = System.currentTimeMillis();
    list.stream().filter(it -> it > 300).map(String::valueOf);
    System.out.println(System.currentTimeMillis() - mills1);
    输出结果为 116， 加上 collect(Collectors.toList()) 为 198


## SkipList 跳表
对标的是平衡树，insert/delete/search 都是 O(log n)

底层是链表，链表无法像arraylist一样做到快速查询，便使用类似二分法的方式做索引。
    
    比如说，9个元素，第一层的索引是0 -> 4 -> 8，锁定两个区间
    第二层索引是 0 -> 2 -> 4 再锁定两个区间
    从而在链表上实现二分查找的效果

## List

List.stream().foreach 与 List.foreach 的区别，前者无序后者有序。前者使用的Spliterators，后者使用的Iterators。


contains, remove之类调用的比较都是equals。
在使用lambok的@Data时，由于会重写equals方法，在下面case下会有异常

    class A {
        private String name;
    }
    @Data
    class B extends A {
        private String value;
    }
    public static void main(String...args) {
        A a = new B();
        A b = new B();
        a.setName("name");
        a.equals(b);    // true
    }
    lombok的@Data生成equals方法时，默认只比较该类自身定义的各个属性值是否一致。在上例中，value都是null，因而判断为true。添加 @EqualsAndHashCode(callSuper=true)或者不适用@Data则可以解决
    
clone方法在object类中native实现

### Spliterators
方法主要有 tryAdvance trySplit forEachRemaining

## Map
### HashMap
#### 1.8
##### hash计算
通过key的hashCode，前16位&后16位，之后和当前的数组长度-1（2的x次方-1）进行&操作。
##### get
计算hash，从数组[hash]上的链表或者红黑树上使用==或equals找目标

Q: 在多线程的情况下，在链表上（或红黑树）查询时，线程A时间片耗尽让出cpu，另一个线程B进行了put并触发了resize()，导致链表（或红黑树）因为hash发生变化出现了重构,A当前链表（红黑树）后面的部分节点到了另一个链表（红黑树）上，从而取不到该有的值？
##### put
1. 计算hash，在数组上找到需要插入的位置。
2. 数组为空时初始化，位置上为空时直接插入
3. 为树时插入树节点，为链表时插入链表节点（长度>=8时进行树化）
4. 判断是否需要resize()
##### resize
1. 数组扩大一倍
2. 判断当前下标是否为空或只有一个节点，直接插入新数组
3. 对于链表，通过 hash & 原数组大小的结果判断是否在原位置，移动链表的元素。对于树，TreeNode继承自LinkedHashMap.Entry，本身就是有序的，遍历，生成链表，之后再看是否树化。

### ConcurrentHashMap
1.7使用分段锁
1.8使用synchronized加CAS,只锁数组上的node，如果不发生hash碰撞就全是无竞争并发

### TreeMap


