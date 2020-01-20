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

## List

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

