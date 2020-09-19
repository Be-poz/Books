본 내용은 Head First Design Patterns 책을 토대로 정리하였습니다.

날씨 정보를 받아서 각 디스플레이에 출력을 해야한다고 가정하자.
```java
public class WeatherData{
    
    public void measurementsChanged(){
        float temp = getTemperature();
        float humidity = getHumidity();
        float pressure = getPressure();
    }
    
    getTempreature();
    getHumidity();
    getPressure();
 
    currentConditionsDisplay.update(temp,humidity,pressure);    
    statisticsDisplay.update(temp,humidity,pressure);
    forecastDisplay.update(temp,humidity,pressure);
 
}

```
대충 다음과 같다고 하자.

이 코드에서는 구체적인 구현에 맞춰서 코딩했기 때문에 프로그램을 고치지 않고는 다른 디스플레이 항목을

추가/제거 할 수 없다. 그리고 모두 똑같은 파라미터를 가진 update() 메서드를 가지고 있다.

옵저버 패턴은 신문이나 잡지를 구독하는 것과 같은 매커니즘이다.

주제(subject) 객체가 옵저버(observer) 객체를 관리한다. (신문사가 구독자들을 관리한다)

신규 구독자가 구독하고 싶으면 옵저버 무리에 추가하고 해지하면 제거한다.

**즉, 옵저버 패턴에서는 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고 자동으로 내용이 갱신되는 방식으로 일대다(one-to-many) 의존성을 정의한다.**

![observerImg1](https://user-images.githubusercontent.com/45073750/93667689-ce4d6700-fac2-11ea-8784-f038fb81979d.png)

Subject / Observer 인터페이스를 각각 구현해서 표현하였다.

옵저버가 될 가능성이 있는 객체에서는 반드시 Observer 인터페이스를 구현해야 한다. 인터페이스에는 주체의 상태를 바뀌었을 때 호출되는 update() 메서드 밖에 없다.

주제 역할을 하는 구상 클래스에서는 항상 Subject 인터페이스를 구현해야 한다. 주제 클래스에는 등록 및 해지를 위한 메서드 외에 상태가 바뀔 때마다 모든 옵저버들에게 연락을 하기위한 notifyObserver() 메서드도 구현해야 한다.

두 객체는 **느슨한 결합**으로 이루어져 있다.

1\. 주제가 옵저버에 대해서 아는 것은 옵저버가 특정 인터페이스를 구현한다는 것 뿐이다.

2\. 옵저버는 언제든지 새로 추가할 수 있다.

3\. 새로운 형식의 옵저버를 추가하려고 할 때에도 주제를 변경할 필요가 없다.

4\. 주제와 옵저버는 서로 독립적으로 재사용할 수 있다.

5\. 주제나 옵저버가 바뀌더라도 서로한테 영향을 미치지 않는다.

각 디스플레이에서 보여줄 값들은 그 내용이 다르니 display() 메서드를 적절히 추가해 주어야 한다.

![observerImg2](https://user-images.githubusercontent.com/45073750/93667711-e91fdb80-fac2-11ea-8bb8-cdce2dc111a9.png)

다음과 같이 클래스 다이어그램을 작성해보았다. 

이제 코드로 구현해보자

```java
public interface Subject{
    public void registerObserver(Observer o);
    public void removeObserver(Observer o);
    public void notifyObserver(Observer o);
}
 
public interface Observer{
    public void update(float temp, float humidity, float pressure);
}
 
public interface DIsplayElement{
    public void display();
}

```

interface 들은 다음과 같다.

```java
public class WeatherData implements Subject{
    private ArrayList observers;
    private float temperature;
    private float humidity;
    private float pressure;
    
    public WeatherData(){
        observers = new ArrayList();
    }
 
    public void registerObserver(Observer o) {
        observers.add(o);
    }
 
    public void removeObserver(Observer o) {
        int i=observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }
    
    public void notifyObservers(){
        for (int i = 0; i < observers.size(); i++) {
            Observer observer=(Observer)observers.get(i);
            observer.update(temperature, humidity, pressure);
        }
    }
    
    public void measurementsChanged() {
        notifyObservers();
    }
 
    public void serMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();;
    }    
}

```

Subject 인터페이스를 WeatherData에서 구현을 한 코드다.

옵저버 등록 삭제 알리는 코드들을 구현했다.

값들이 set 되면 측정값이 바뀌었다는 MeasurementsChanged() 가 호출되고 옵저버들에게 그 값들을 알리게 된다.

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement{
    private float temperature;
    private float humidity;
    private Subject weatherData;
 
    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }
 
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }
    
    public void display() {
        System.out.println();
    }
}
```

display 항목을 만들었다.

weatherData에 옵저버 등록을 하고 update()메서드를 작성하였다.

```java
public class WeatherStation{
    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();
 
        CurrentConditionsDisplay currentConditionsDisplay = new CurrentConditionsDisplay(weatherData);
        StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);
        
        weatherData.serMeasurements(80,65,30,4f);
    }
}
```

다음과 같이 실행할 수가 있다.

지금까지는 이제 Subject가 Observer한테 푸쉬하는 방식이었다. 이를 풀 방식으로도 구현할 수가 있다.

지금까지 직접 구현을 하였지만 자바는 자체적으로 옵저버 패턴을 지원한다.

가장 일반적으로 쓸 수 있는 것은 java.util 패키지에 있는

Observer 인터페이스와 Observable 클래스이다. 

![observerImg3](https://user-images.githubusercontent.com/45073750/93667714-f210ad00-fac2-11ea-9db3-9b220b9fadb6.png)

다음과 같이 이루어져 있다.

Observable 클래스에서 모든 옵저버들을 관리하고 우리 대신 연락을 해준다. setChanged()가 추가되었다.

DisplayElement 인터페이스를 생략했지만, 디스플레이 항목에서 그 인터페이스를 구현해야 하는건 마찬가지이다.

주제 클래스인 WeatherData에서는 더 이상 register, remove, notify 와 같은 메서드가 필요 없다. 그냥 슈퍼 클래스에서 상속 받으면 되니깐 말이다.

이 상황에서도 객체가 옵저버가 되기 위해서는 Observer 인터페이스를 구현하고 Observable 객체의 addObserver() 메서드를 호출하면 된다.

Observable 에서 연락을 돌릴 때에는

1\. setChanged() 메서드를 호출해서 객체의 상태가 바뀌었다는 것을 알린다.

2\. notifyObservers() 또는 notifyObservers(Object arg) 를 호출한다.

옵저버가 연락을 받을 때에는

update(Observable o, Object arg) 를 통해 받는다. 

연락을 받는 주제 객체가 인자로 전달이 되고, notifyObservers() 메서드에서 인자로 전달된 데이터 객체, 만약 지정되지 않은 경우에는 null 값을 인자로 준다.

이것은 푸쉬이고 그렇다면 풀은 어떻게 해야할까??

그전에 setChanged() 메서드가 어쩌다 추가되었는지 알아보자

```java
   setChanged() {
        changed = true;
    }
 
    notifyObservers(Object arg) {
        if(changed){
            update(this, arg);
        }
        changed=false;
    }
    
    notifyObservers(){
        notifyObservers(null);
    }
```

setChanged() 메서드는 상태가 바뀌었다는 것을 밝히기 위한 용도로 쓰인다.

왜 필요할까?

setChanged() 메서드는 연락을 최적화할 수 있게 해준다.

만약 기상스테이션의 온도 센서가 민감해서 0.1도 단위로 쉴 새 없이 값이 바뀐다고하자, 하지만 필요에 따라 온도가

0.5도 이상 바뀌었을 때만 연락을 돌리고 싶을 수도 있을텐데, 이 때에 setChanged() 메서드를 조건에 따라 호출함으로써 원하는 바를 달성할 수가 있는 것이다.

자바가 지원하는 방식으로 코드를 구현해보자

```java
import java.util.Observable;
import java.util.Observer;
 
public class WeatherData extends Observable{
    private float temperature;
    private float humidity;
    private float pressure;
 
    public WeatherData(){}
 
    public void measurementsChanged(){
        setChanged();
        notifyObservers();
    }
 
    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
 
    public float getTemperature() {
        return temperature;
    }
 
    public float getHumidity(){
        return humidity;
    }
 
    public float getPressure() {
        return pressure;
    }
}
```

코드를 보면 setMeasurements() 메서드가 발생하면 measurementsChanged() 메서드가 호출되고

이 안에서 setChanged()와 notifyObservers()가 호출이 된다. 위에서 setChanged() 조건을 달 수 있다고 했다.

원한다면 조건을 추가하면 될 것이다. 그리고 notifyObservers() 에서 어떤 데이터 객체도 보내고 있지 않다.

여기에서는 풀 모델을 사용하고 있기 때문이다. 

getter를 추가한 것은 옵저버에서 풀로 땡겨올 때 사용하기 위함이다.

이제 옵저버 부분도 구현해보자

```java
import java.util.Observable;
import java.util.Observer;
 
public class CurrentConditionsDisplay implements Observer, DisplayElement{
    Observable observable;
    private float temperature;
    private float humidity;
 
    public CurrentConditionsDisplay(Observable observable) {
        this.observable = observable;
        observable.addObserver(this);
    }
 
    public void update(Observable obs, Obejct args) {
        if (obs instanceof WeatherData) {
            WeatherData weatherData = (WeatherData)obs;
            this.temperature = weatherData.getTemperature();
            this.humidity = weatherData.getHumidity();
            display();
        }
    }
    
    public void display(){
        System.out.println();
    }
}
```

다음과 같이 옵저버에서 주제에 대해 풀을 하여 정보를 가져왔다.

이런식으로 옵저버 패턴을 사용할 수가 있다.
