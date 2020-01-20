# Design Patterns
## 观察者模式

    interface Subject {
        void addObserver(Observer observer);

        void removeObserver(Observer observer);

        void notifyObservers();
    }

    interface Observer {
        void update();
    }

    static class WeatherData implements Subject {
        List<Observer> observers = Lists.newArrayList();
        int temperate;

        @Override
        public void addObserver(Observer observer) {
            observers.add(observer);
        }

        @Override
        public void removeObserver(Observer observer) {
            observers.remove(observer);
        }

        @Override
        public void notifyObservers() {
            observers.forEach(Observer::update);
        }

        void updateTemperate(int t) {
            this.temperate = t;
            notifyObservers();
        }
    }

    static class WeatherDisplay implements Observer {
        WeatherData weatherData;

        WeatherDisplay(WeatherData weatherData) {
            this.weatherData = weatherData;
            weatherData.addObserver(this);
            update();
        }

        @Override
        public void update() {
            System.out.println(weatherData.temperate);
        }
    }

    public static void main(String... args) {
        WeatherData weatherData = new WeatherData();
        WeatherDisplay display = new WeatherDisplay(weatherData);
        weatherData.updateTemperate(3);
        weatherData.updateTemperate(4);
    }

## 发布订阅模式
发布者与订阅者之间有中间商。
发布者不知道订阅者的存在，将所有的内容都会发布。
中间商会将内容归类，然后发给订阅的用户（或者自取？）。
整个过程可以为异步。
中间商和发布者可以通过消息队列进行通信
中间商和发布者可以使用观察者模式，也可使用消息队列进行通信

## 监听者模式
类似于观察者模式，将主题拆分为事件源和事件。事件源会把事件传递给接收者。
java基础包提供了一个标记性的接口 EventListener, 提供了遗失基础的事件类 EventObject

### EventListener
空接口，标注为监听者
### EventObject
事件
持有一个数据源的引用

### spring 提供的监听者
#### ApplicationListener
继承自 EventListener
标记为函数式接口
提供响应事件的方法 onApplicationEvent
#### ApplicationEventPublisher
标记为函数型接口
提供发布事件的方法 publishEvent
被 ApplicationContext 继承

#### ApplicationEvent
抽象类
继承自 EventObject
申明一个 final 方法获取构造函数初始化时的系统时间

#### AbstractApplicationContext
持有一个 ApplicationEvent 的 set，持有一个 ApplicationEventMulticaster（事件的辅助接口）
set 不为 null 时直接添加，为空时交给 ApplicationEventMulticaster 处理
如果该 context 存在 parent，则最后调 parent 的 publishEvent

容器会在启动的时候将 set 赋值并置空，完成对启动事件的监听。后续业务事件交给辅助接口。

拥有一个 applicationListener 的 set

#### SimpleApplicationEventMulticaster
持有一个内部类 ListenerRetriever 的实例，存有一个 ApplicationListener 的 set
获取 application 的所有listener，并调用他们的 onApplication 方法

### SealedClass
继承受限的类。
枚举如果有 value，必须是 final 的，不然在多线程下会出现问题。
SealedClass 可以解决上述问题，不同的实例拥有不同的 value，但是实际上似乎没有合适的应用场景。

## 封装变化
将会变的和不会变的拆分开来。

会变的如鸭子与游泳，各种鸭子从小就会游泳
不会变的如鸭子与飞行，有的鸭子不一定会飞行
游泳是所有鸭子都具有的行为，因而可以在基类中申明或者实现方法
飞行未必是所有鸭子都会，因而可以申明为一个接口变量

## 面向接口
面向接口编程可以不用在乎实现细节，降低耦合，在变化的时候能够减小对该接口及实现以外的影响


