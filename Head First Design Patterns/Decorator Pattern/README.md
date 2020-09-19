본 내용은 Head First Design Patterns 책을 토대로 정리하였습니다.

![decorator1](https://user-images.githubusercontent.com/45073750/93668404-fc34aa80-fac6-11ea-9042-921f05bc04df.png)

카페에서 다양한 음료를 팔기 위해 주문 시스템을 갖추는 과정에서 음료에 대한 가격을 측정하려고 할 때,

위와 같이 행한다면 우유, 두유, 초콜릿, 크림 등 추가할 때마다 각각의 클래스가 생성될 것이고 그 수는 감당이 안될 정도로 불어날 것이다.

![decorator2](https://user-images.githubusercontent.com/45073750/93668405-fd65d780-fac6-11ea-980c-bbb074ec7cd5.png)

그렇다면 다음과 같이 Beverage클래스에 각 데코레이션을 할 수 있는 재료들을 넣고 hasSoy, setSoy와 같이 메서드를 추가하고 각 첨가된 항목 별로 cost() 메서드를 통해 가격을 측정하면 클래스가 많아지는 것을 방지할 수 있지 않을까?

클래스의 수는 막을 수 있겠지만,

1\. 첨가물 가격이 바뀔 때마다 기존 코드를 수정해야 한다.

2\. 첨가물의 종류가 많아지면 새로운 메서드를 추가해야 하고, 슈퍼클래스의 cost() 메서드를 고쳐야 한다.

등의 문제가 추가로 발생하게 된다.

**클래스는 확장에 대해서는 열려 있어야 하지만 코드 변경에 대해서는 닫혀 있어야 하는**

**OCP(Open-Closed Principle)이라는 디자인 원칙을 적용해야 한다.**

(Subject자체에코드를 추가하지 않았으면서도 확장을 한 옵저버 패턴처럼 말이다.)

이것을 해결하기 위해 특정 음료에서 시작해서, 첨가물로 그 음료를 데코레이트 하는 방식을 취할 것이다.

만약, 어떤 손님이 모카하고 휘핑 크림을 추가한 다크 로스트 커피를 주문한다면

1\. DarkRoast 객체를 가져온다.

2\. Mocha 객체로 장식한다.

3\. Whip 객체로 장식한다.

4\. cost() 메서드를 호출한다.

위의 순서를 따를 것이다. 

DarkRoast 라는 객체를 Mocha가 감쌀 것이다. 이 Mocha 객체를 Whip 객체가 감쌀 것이다.

각 객체 안에는 cost() 메서드가 들어가 있다. 

가격을 구할 때에는 가장 바깥쪽에 있는 데코레이터인 Whip의 cost()를 호출한다. 그러면 Whip 에서는 그 객체가 장식하고 있는 객체한테 가격 계산을 위임하고, 가격이 구해지고 나면 구해진 가격에 휘핑 크림의 가격을 더한 다음 그 결과를 리턴하는 식의 계산을 하게된다.

데코레이터 패턴에서는 객체에 추가적인 요건을 동적으로 첨가한다. 데코레이터는 서브클래스를 만드는 것을 통해서 기능을 유연하게 확장할 수 있는 방법을 제공한다.

![decorator3](https://user-images.githubusercontent.com/45073750/93668406-fe970480-fac6-11ea-8651-a184d0f40777.png)

다음과 같은 클래스 다이어그램의 형태를 보인다.

이 상황의 경우에는

![decorator4](https://user-images.githubusercontent.com/45073750/93668407-ffc83180-fac6-11ea-802d-3283c32418a1.png)

다음과 같이 표현할 수가 있다.

코드로 표현해보면

```java
public abstract class Beverage {
    String description="제목 없음";
    
    public String getDescription() {
        return description;
    }
    
    public abstract double cost();
}
 
 
public abstract class CondimentDecorator extends Beverage{
    public abstract String getDescription();
}
```

다음과 같다. Beverage 클래스는 추상 클래스이며 getDescription() 메서드는 구현이 되었지만 cost() 메서드는 서브클래스에서 구현해야 한다.

첨가물을 나타내는 추상 클래스 CondimentDecorator 에서는 모든 첨가물 데코레이터에서 getDescription() 메서드를

새로 구현하도록 추상 메서드로 선언하였다.

```java
public class Espresso extends Beverage{
    
    public Espresso() {
        description = "에스프레소";
    }
    
    public double cost(){
        return 1.99;
    }
}
 
public class HouseBlend extends Beverage{
 
    public HouseBlend() {
        description = "하우스 블렌드 커피";
    }
 
    public double cost(){
        return 0.99;
    }
}
```

음료 코드를 구현하였다. Beverage 클래스에서 받은 description 변수에 이름을 설정하였다.

cost() 에서는 가격만 리턴하였다. 첨가물 가격을 더하는 건 이곳에서 하지 않는다.

```java
public class Mocha extends CondimentDecorator {
    Beverage beverage;
 
    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }
    
    public String getDescription() {
        return beverage.getDescription() + ", 모카";
    }
    
    public double cost() {
        return 0.20 + beverage.cost();
    }
}
```

첨가물용 코드를 구현하였다. 생성자를 통하여 감싸고자 하는 음료 객체를 전달 받았다.

이름에는 감싼 이름에다가 해당 첨가물의 이름을 더했다.

감싸진 음료의 가격에 Mocha의 가격을 더한 값을 return 해준다.

이제 전체적으로 코드를 돌려보자

```java
public class test{
    public static void main(String[] args) {
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription()+" "+beverage.cost());
 
        Beverage beverage2 = new HouseBlend();
        beverage2 = new Mocha(beverage2);
        System.out.println(beverage2.getDescription()+" "+beverage2.cost());
    }
}
 
에스프레소 1.99
하우스 블렌드 커피, 모카 1.19
```

의도한 대로 출력이 잘 되는 것을 볼 수가 있다.

Q: 만약 특정 구상 구성요소인지 확인한 다음 어떤 작업을 처리하는(하우스 블렌드 커피만 특별 할인하는) 경우에는 이 코드를 사용할 수 없지 않나요? HouseBlend가 데코레이터로 감사찌고 나면 그 커피가 하우스 블렌드인지 알 수 없으니깐요??

A: 맞습니다. 구상 구성요소의 형식을 알아내서 그 결과를 바탕으로 어떤 작업을 처리하는 코드에 데코레이터 패턴을 적용하면 코드가 제대로 돌아가지 않습니다. 그러한 코드를 만들어야 한다면 데코레이터 패턴을 사용하는 것에 대해 다시 생각해볼 필요가 있습니다.

Q: 데코레이터가 같은 객체를 감싸고 있는 다른 데코레이터에 대해서 알 수 있나요? 예를 들어서 getDescription() 메서드에서 "모카, 휘핑 크림, 모카" 라고 출력하는 대신 "휘핑 크림, 더블 모카" 와 같은 식으로 출력할 수 있게 할 수 있나요? 그러려면 가장 바깥쪽의 데코레이터가 내부의 데코레이터들을 전부 알 수 있어야 하잖아요

A: 위의 질문과 유사한데, 데코레이터는 그 데코레이터가 감싸고 있는 객체에 행동을 추가하기 위한 용도로 만들어진 것이기에 여러 단계의 데코레이터를 파고 들어가서 어떤 작업을 해야한다면, 원래 데코레이터 패턴이 만들어진 의도와 어긋나는 것입니다. 하지만, 굳이 해결하자면 description 변수를 파싱하거나 해서 이름을 바꾸면 되긴 합니다. getDescription() 메서드 내에 ArrayList를 리턴하도록 한다면 더욱 쉬워질 것입니다. (위의 할인의 경우에도 적용 가능할 것 같습니다.)

자바에도 데코레이터 패턴이 적용된 예가 있다. 그것은 바로 **java.io **패키지이다.

![decorator5](https://user-images.githubusercontent.com/45073750/93668408-00f95e80-fac7-11ea-8934-23e6fd78a6a5.png)

대략 다음과 같이 구성되어 있다.

이제 이 데코레이터를 이용해서 직접 입력 데코레이터를 만들어보자

```java
public class test{
    public static void main(String[] args) throws IOException {
        int c;
        try {
            InputStream in =
               new LowerCaseInputStream(
                    new BufferedInputStream(
                            new FileInputStream("wow.txt")));
            while ((c = in.read()) >= 0) {
                System.out.print((char) c);
            }
            in.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
 
class LowerCaseInputStream extends FilterInputStream {
    public LowerCaseInputStream(InputStream in) {
        super(in);
    }
 
    public int read() throws IOException{
        int c = super.read();
        return (c == -1 ? c : Character.toLowerCase((char) c));
    }
 
    public int read(byte[] b, int offset, int len) throws IOException {
        int result = super.read(b, offset, len);
        for (int i = offset; i < offset + result; i++) {
            b[i] = (byte) Character.toLowerCase((char) b[i]);
        }
        return result;
    }
}
```

이제 평소에 사용하는 InputStream의 파악도 됐을 것이다.