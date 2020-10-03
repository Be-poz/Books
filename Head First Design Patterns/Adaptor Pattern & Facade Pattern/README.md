# Adaptor Pattern & Facade Pattern
본 내용은 Head First Design Patterns 책을 토대로 정리하였습니다.

## 어댑터 패턴(Adaptor Pattern)

어댑터 패턴은 기존 시스템에서 사용하는 인터페이스와 요구되는 인터페이스가 다를 때에 중개해주는 역할을 한다.  

다음과 같은 예시가 있다.  
```java
public interface Duck{
    public void quack();
    public void fly();
}

public class MallardDuck implements Duck{
    public void quack(){
        System.out.println("Quack");
    }
    public void fly(){
        System.out.println("I'm flying");
    }
}

public interface Turkey{
    public void gobble();
    public void fly();
}

public class WildTurkey implements Turkey{
    public void gobble(){
        System.out.println("Gobble gobble");
    }
    public void fly(){
        System.out.println("I'm flying a short distance");
    }
}


public class TurkeyAdaptor implements Duck{
    Turkey turkey;

    public TurkeyAdaptor(Turkey turkey) {
        this.turkey = turkey;
    }
    
    public void quack() {
        turkey.gobble();
    }
    public void fly() {
        for (int i = 0; i < 5; i++) {
            turkey.fly();
        }
    }
}

public class DuckTestDrive{
    public static void main(String[] args) {
        MallardDuck duck = new MallardDuck();

        WildTurkey turkey = new WildTurkey();
        Duck turkeyAdaptor = new TurkeyAdaptor(turkey);

        System.out.println("turkey says");
        turkey.gobble();
        turkey.fly();

        System.out.println("duck says");
        duck.quack();
        duck.fly();

        System.out.println("turkeyAdaptor says");
        duck.quack();
        duck.fly();
    }
}
```
아주 긴 예시긴 하지만 이해가 바로 될 것이다.  

>**어댑터 패턴** - 한 클래스의 인터페이스를 클라이언트에서 사용하고자 하는 다른 인터페이스로 변환한다. 어댑터를 이용하면 인터페이스 호환성 문제 때문에 같이 쓸 수 없는 클래스들을 연결해서 쓸 수 있다.  

![adaptor1](https://user-images.githubusercontent.com/45073750/94987898-9525f400-05a4-11eb-8307-d600f7ac84c0.PNG)
클래스 다이어그램은 다음과 같다.  
위의 코드에서는 Target이 Duck인터페이스가 되고 Adaptee는 Turkey 객체가 된다.  
클라이언트는 Duck 한테 얘기하고 있다고 생각한다.  
Adaptor에서는 Duck 인터페이스를 구현하지만, 메서드가 호출되었을 때 그 호출을 Turkey메서드 호출로 변환해 준다.  
Adaptor 덕분에 클라이언트에서 Duck 인터페이스에 대해 호출한 것을 Turkey에 대해서도 받아서 처리할 수 있게 된다.  

자바의 `Iterator` 인터페이스가 대표적인 어댑터 패턴의 예이다.  
***
 ## 퍼사드 패턴(Facade Pattern)  
 
|패턴|용도|  
|:---:|:---:|
|데코레이터|한 인터페이스를 다른 인터페이스로 변환|
|어댑터|인터페이스는 바꾸지 않고 책임(기능)만 추가|
|퍼사드|인터페이스를 간단하게 바꿈|  

홈 씨어터를 사용하려고 한다. 그런데 해야할 일이 정말 많다. 코드로 나타내면 가령 다음과 같다고 하자.  
```java
popper.on();
popper.pop();

lights.dim(10);

screen.down();

projector.on();
projector.setInput(dvd);
projector.wideScreenMode();

amp.on();
amp.setDvd(dvd);
amp.setSurroundSound();
amp.setVolume(5);

dvd.on();
dvd.play(movie);
```
너무나도 복잡한데 퍼사드 패턴을 사용하면 훨씬 쓰기 쉬운 인터페이스를 제공할 수가 있다.  
HomeTheaterFacade 라는 클래스에서 홈 씨어터 구성요소들을 호출할 것이다.  

>Q: 퍼사드에서 서브시스템 클래스들을 캡슐화하면 저수준 기능을 원하는 클라이언트에서는 어떻게 서브시스템 클래스에 접근하나요?  
>A: 퍼사드 클래스에서는 서브시스템 클래스들을 캡슐화하지 않기 때문에 필요하다면 서브시스템 클래스를 그냥 사용하면 된다.  
>
>Q: 간단한 인터페이스를 만들 수 있다는 장점 외에는 없나요?  
>A: 클라이언트 구현과 서브시스템을 분리시킬 수 있다. 만약 홈 씨어터를 업그레이드 한다면 퍼사드 클래스만 바꾸면 될 것이다.  

```java
public class HomeTheaterFacade {
    Amplifier amp;
    Tuner tuner;
    DvdPlayer dvd;
    CDPlayer cd;
    Projector projector;
    TheaterLights lights;
    Screen screen;
    PopcornPopper popper;
    
    public HomeTheaterFacade(...){
        ...
    }

    public void watchMovie(String movie) {
        popper.on();
        popper.pop();
        ...
        dvd.on();
        dvd.play(movie);
    }
    
    public void endMovie() {
        popper.off();
        dvd.stop();
        dvd.ejct();
        dvd.off();
    }
}
```
다음과 같이 사용할 수가 있다.  

>**퍼사드 패턴** - 어떤 서브시스템의 일련의 인터페이스에 대한 통합된 인터페이스를 제공한다. 퍼사드에서 고수준 인터페이스를 정의하기 때문에 서브시스템을 더 쉽게 사용할 수 있다.  

퍼사드 패턴은 다른 패턴과 달리 추상화도 사용하지 않고 단순하다. 그렇다고 별 볼 일 없는 패턴이라고는 할 수 없다.  

모든 패턴을 사용하고, 객체를 만듦에 있어 **최소 지식 원칙**을 따르는 것을 지향해야 한다.  

어떤 메서드에서든지 다음 네 종류의 객체의 메서드만을 호출하면 된다.  
* 객체 자체
* 메서드에 매개변수로 전달된 객체
* 그 메서드에서 생성하거나 인스턴스를 만든 객체
* 그 객체에 속하는 구성요소  

```java
public float getTemp(){
    Thermometer thermometer = station.getThermometer();
    return thermometer.getTemperature();
}

public float getTemp(){
    return station.getTemperature();
}
```

위의 코드가 아래의 코드로 원칙을 따라서 바뀌었다.  
station 클래스에 thermometer에 요청을 해주는 메서드를 추가해준 덕분이다.  

다음과 같이 **최조 지식 원칙** 을 지킬 수가 있다.  
***