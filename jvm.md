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

    LinkedList.remove 方法，会在循环中将 node 的每个属性都置为 null，再将 node 置为 null
    ArrayList.remove 方法，会在循环中将数组的每个成员都置为 null

#### ThreadLocalMap
threadLocalMap（没有实现 map 接口） 的 entry 继承自 weakReference

    构造器：
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            ...
        }
        
gc 时，key 会回收，而 value 不会。回收后在调用 threadLocal.get 时会触发清除（threadLocalMap.expungeStaleEntry)。

#### Cleaner
DirectByteBuffer(能够申请堆外内存）持有一个Cleaner，在对象回收后释放堆内存。
Cleaner为一个虚引用，继承 PhantomReference
        
## Class
### 加载过程
Loading -> Verification -> Preparation -> Resolution -> Initialization -> Using -> Unloading
预加载过程中遇到.class文件缺失或者存在错误，直到首次使用才会报告错误
#### Loading
1.通过一个类的全限定名来获取其定义的二进制字节流
2.将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3.在java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口

#### Verification
1.文件格式验证
2.源数据验证
3.字节码验证
4.符号引用验证

该阶段不是必须的， - Xverifynone 参数可以关闭大部分类验证措施
#### Preparation
为类的静态变量分配内存、初始化为零值
#### Resolution
把类中的符号引用转换为直接引用
#### Initialization
为类的静态变量赋予正确的初始值

步骤：

    1.类未加载和连接，先加载和连
    2.直接父类未初始化，先初始化其直接父类
    3.类中有初始化语句，依次执行    
    
时机：
    主动使用
    
    1.创建类的实例
    2.访问类或接口的静态变量或对其赋值
    3.调用类的静态方法
    4.使用反射
    5.java虚拟机标明为启动的类，或者java.exe直接执行的主类
    
#### Unloading
1.执行System.exit()
2.程序正常执行结束
3.执行过程中遇到了异常或错误
4.操作系统发生错误导致虚拟机进程终止

#### ClassLoader
##### 种类
1.启动类加载器（BootstrapClassLoader),负责加载JDK/jre/lib或被 - Xbootclasspath 指定的路径中能被虚拟机识别的类库，由c++
实现，无法用户调用
2.扩展类加载器(ExtensionClassLoader)加载 JDK/jre/lib/ext目录或者 java.ext.dirs系统变量指定的路径的类库
3.应用程序类加载器(ApplicationClassLoader)加载用户 classpath 索引定的类，为程序的默认类加载器
##### 机制
1.全盘负责，类加载器加载Class时，该Class索引来和引用的没显示指定类加载器的其他Class都由该类加载器载入
2.父类委托，先尝试让父类加载该类，失败后自行加载
3.缓存机制，所有加载过的Class都会被缓存，只有缓存区不存在时才读取类对应的二进制文件，并将其转换为Class对象，存入缓存区

#### 类加载方式
1.命令行启动应用时由JVM初始化加载
2.通过Class.forName()动态加载
3.通过ClassLoader.loadClass()方法动态加载


