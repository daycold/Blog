# REFLECTION
## 泛型
反射时，只能拿到泛型的上界，而不会得到具体的类型。
只会在类定义信息中获取，而不能动态的在调用时获取。

      fun <T : Func> doSth(): T
      
      method.returnType.simpleName: "Func"

## Method
suspend方法在使用java反射时会多一个协程的参数。使用kotlin反射的function时不会有。
### genericParameterTypes
返回的参数Type数组保留了泛型信息
（is AnnotatedType）
### declaringClass
获取的方法申明的位置。从所在类开始沿继承链向上查找，找到第一个（可以是重写的方法）
动态代理会返回代理的名称

    println(b::class.java.getMethod("doA").declaringClass.simpleName)
    println(c::class.java.getMethod("doA").declaringClass.simpleName)
    println(B::class.java.getMethod("doA").declaringClass.simpleName)

    其中 b 为B接口的动态代理实现，c为B接口的直接实现，B接口继承A接口，A接口申明doA方法
    返回：
    $Proxy0
    C
    A
    
在实现接口方法的时候回申明方法，所以 b 和 c 两个对象打印出自己的类名
B 在重复申明 doA 方法后打印的 B，不重复申明打印的 A

## Annotation
使用 AnnotatedElementUtils等spring的工具类才能拿到@AliasFor的数据

