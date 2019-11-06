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

