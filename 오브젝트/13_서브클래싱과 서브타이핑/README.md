서브클래싱과 서브타이핑
-

<h3> 타입 </h3>

프로그래밍 언어에서 타입은 2가지 목적을 위해 사용된다.
1. 타입에 수행될 수 있는 유효한 오퍼레이션의 집합을 정의  
ex) 자바에서 '+'는 숫자 타입이나 문자열 타입 객체에는 사용가능하지만 다른 클래스의 인스턴스에 대해서는 불가능하다.
객체의 타입에 따라 적용 가능한 연산자의 종류를 제한함으로써 실수를 막아준다.  

2. 타입에 수행되는 오퍼레이션에 대해 미리 약속된 문맥을 제공  
ex) a+b 연산에서 a와 b가 int라면 두 수를 더할 것이고, string이라면 문자열을 합칠 것이다. 따라서 a와 b에 부여된 타입이 '+' 연산자의 문맥을 정의한다.  
***

<h3> 타입 계층 </h3>

'프로그래밍 언어' 라는 타입안에 '자바', 'C', '자바스크립트' 등이 존재하고, 더 세분화한다면 '프로그래밍 언어' 타입은 '객체지향 언어' 타입과
'절차적 언어' 타입을 포함하고, '객체지향 언어' 내에는 '클래스 기반', '프로토타입 기반' 이렇게 나뉠 수 있을 것이다.  

이를 일반화와 특수화 관계를 가진 계층으로 표현한다면, 타입 계층을 표현할 때는 더 일반적인 타입을 위쪽에, 더 특수한 타입을 아래쪽에 배치하는 것이 관례이다.  

더 일반적인 타입을 `슈퍼타입` 더 특수한 타입을 `서브타입` 이라고 부른다.  
'프로그래밍 언어'타입은 '객체지향 언어', '절차적 언어' 타입의 `슈퍼타입`이고,  
'객체지향 언어'타입은 '클래스 기반 언어', '프로토타입 기반 언어'의 `슈퍼타입`이라고 볼 수 있다.  

`슈퍼타입`은
* 집합이 다른 집합의 모든 멤버를 포함한다.
* 타입 정의가 다른 타입보다 더 일반적이다.

`서브타입`은
* 집합에 포함되는 인스턴스들이 더 큰 집합에 포함된다.
* 타입 정의가 다른 타입보다 좀 더 구체적이다.  

이것을 객체지향 프로그래밍 관점에서 정의 한다면 다음과 같다.
* `슈퍼타입`이란 `서브타입`이 정의한 퍼블릭 인터페이스를 일반화시켜 상대적으로 범용적이고 넓은 의미로 정의한 것이다.
* `서브타입`이란 `슈퍼타입`이 정의한 퍼블릭 인터페이스를 특수화시켜 상대적으로 구체적이고 좁은 의미로 정의한 것이다.
***

<h3> 서브클래싱과 서브타이핑 </h3>

상속을 이용해 타입 계층을 구현한다는 것은 부모 클래스가 `슈퍼타입`의 역할을, 자식 클래스가 `서브타입`의 역할을 수행하도록 클래스 사이의 정의를 한다는 것이다.  

상속의 올바른 용도는 타입 계층을 구현하는 것이다. 다음의 2가지 질문에 모두 '예'라고 답할 수 있을 때에만 사용하라고 '마틴 오더스키'는 조언했다.  
* 상속 관계가 `is-a` 관계를 모델링하는가?
* 클라이언트 입장에서 부모 클래스의 타입으로 자식 클래스를 사용해도 무방한가?  
(상속 계층을 사용하는 클라이언트의 입장에서 부모 클래스와 자식 클래스의 차이점을 몰라야하고 이를 자식과 부모클래스 사이의 `행동 호환성`이라 부른다.)

`is-a` 관계는 사람의 말에 혼동이 되어서는 안된다.  
ex) 펭귄은 새다, 새는 날 수 있다. => 펭귄 is a 새, 라고 하기에는 펭귄은 날 수 없다. 해당 행동을 포함할 수 없기에 서브타입이 될 수 없다.  

위의 예시를 통해 두 타입 사이에 행동이 호환될 경우에만 타입 계층으로 묶어야 한다는 것을 알았다. 행동의 호환 여부를 판단하는 기준은 **클라이언트의 관점**이다.  

```java
public void flyBird(Bird bird) {
    bird.fly()
}

public class Penguin extends Bird{
    @OVerride
    public void fly(){}
}

public class Penguin extends Bird{
    @Override
    public void fly(){
        throw new UnsupportedOperationException();
    }
}

public void flyBird(Bird bird){
 if(!(bird.instanceof Penguin)){
     bird.fly();
 }
}
```
여러 방법으로 문제를 해결해보려 했다.  
1. fly()를 비워두었다. 그러나 클라이언트는 bird가 날 수 있다는 기대를 충족시키지 못하므로 올바른 타입계층이 아니다.
2. 예외를 던졌다. 그러나 모든 새가 날 수 있는데 fly() 로 예외가 던져질 것이라고 기대안했을 것이므로 이 또한 올바르지 않다.
3. instaceof 를 사용했으나 이것은 new 와 같이 클래스에 대한 결합도를 높이고, 타입 추가시마다 코드 수정을 요구하기에 개방-폐쇄 원칙을 위반한다.  

![13-1](https://user-images.githubusercontent.com/45073750/93956765-de4c9b80-fd8d-11ea-9bee-63f46564669f.PNG)
다음과 같이 수정하였다. 문법상으로는 Penguin이 Bird를 상속받더라도 문제가 되지 않지만 Penguin의 퍼블릭 인터페이스에 fly 오퍼레이션이 추가되기 때문에
이 방법을 사용하지 않고 합성을 사용할 것이다.  

사람들은 상속을 사용하는 2가지 목적에 특별한 이름을 붙였는데 `서브클래싱`과 `서브타이핑` 이 그것이다.  
* `서브클래싱`: 다른 클래스의 코드를 재사용할 목적으로 상속을 사용하는 경우를 가리킨다. 자식 클래스와 부모 클래스의 행동이 호환되지 않기 때문에 자식 클래스의 인스턴스가 부모 클래스의 
인스턴스를 대체할 수 없다. **구현 상속** 또는 **클래스 상속** 이라고 부르기도 한다.  
* `서브타이핑`: 타입 계층을 구성하기 위해 상속을 사용하는 경우를 가리킨다. 자식 클래스와 부모 클래스의 행동이 호환되기 때문에 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 있다.
**인터페이스 상속** 이라고 부르기도 한다.  

`서브타이핑` 관계가 유지되기 위해서는 `서브타입`이 `슈퍼타입`이 하는 모든 행동을 동일하게 할 수 있어야 한다. 
즉, 어떤 타입이 다른 타입의 `서브타입`이 되기 위해서는 `행동 호환성`을 만족시켜야 한다.  

자식 클래스와 부모 클래스 사이의 `행동 호환성`은 부모 클래스에 대한 자식 클래스의 `대체 가능성`을 포함한다.
***

<h3> 리스코프 치환 원칙 </h3>

>서브타입은 그것의 기반 타입에 대해 대체 가능해야 한다.  
>클라이언트가 차이점을 인식하지 못한 채 기반 클래스의 인터페이스를 통해 서브클래스를 사용할 수 있어야 한다.

위의 예시인 Penguin과 Bird는 리스코프 치환 원칙을 위반한다.  

이번에 볼 예제는 "정사각형은 직사각형이다." 라는 것이 사실 아니라는 것을 보여주겠다.  

```java
public class Rectangle{
    private int x,y,width, height;

    public Rectangle(int x, int y, int width, int height) {
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }
    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    public Square(int x, int y, int size) {
        super(x,y,size,size);
    }

    @Override
    public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setWidth(height);
        super.setHeight(height);
    }
}

public void resize(Rectangle rectangle, int width, int height){
    rectangle.setWidth(width);
    rectangle.setHeight(height);
    assert rectangle.getWidth() == width&&rectangle.getHeight()==height;
}
```
다음의 상황에서 정사각형은 Rectangle 객체를 상속받았지만 resize 메서드를 하는 과정에서 setWidth와 setHeight으로 인해 너비와 높이가 서로 동일하지 않게되어
Square는 Rectangle이 아니다. 그저 구현을 재사용하고 있었을 뿐이다. 이 두 클래스는 리스코프 치환 원칙을 위반하기 때문에  
`서브타이핑`관계가 아니라 `서브클래싱`관계이다. 이 예시는 `is-a` 관계가 우리의 직관을 얼마나 벗어나게 하는지 보여준다.  

위의 예제를 통해 상속이 `서브타이핑`을 위해 사용될 경우메나 `is-a` 관계인 것을 알 수가 있다.  
***