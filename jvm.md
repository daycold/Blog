# JVM
## reference
强引用: 普通的引用，不会被回收
软引用: SoftReference 对象，可以和 ReferenceQueue 联合使用，内存不足时被回收，可做内存敏感的缓存
弱引用: WeakReference 对象，只有弱引用的对象将会在垃圾回收时回收，可以将传入 WeakReference 构造器的引用置为 null，使对象只有一个弱引用，短时间内使用，在垃圾回收的时候直接回收
虚引用: 用来跟踪对象回收进程的引用

    String a = new String("s")// a 是该对象的一个强引用
    a = null // a 引用失效，上述对象引用数为 0
    
    String a = new String("s");
    SoftReference sr = new SoftReference(a); // 创建一个软引用，该对象有 a, sr 两个引用
    a = null; // 该对象只剩一个软引用，内存不足时会被回收
    
    String a = new String("s");
    WeakReference wr = new WeakReference(a); // 创建一个弱引用，该对象有 a, wr 两个引用
    a = null; // 该对象只剩一个弱引用，垃圾回收时会被回收

