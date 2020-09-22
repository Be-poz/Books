합성과 유연한 설계
-

<h3> 상속을 합성으로 변경하기 </h3>

상속 관계는 `is-a` 관계  
합성 관계는 `has-a` 관계  

상속은 부모 클래스의 내부 구현에 대해 상세하게 알아야 하기 때문에 자식 클래스와 부모 클래스 사이의 **결합도가 높아**질 수 밖에 없다.  
합성은 퍼블릭 인터페이스에 의존해서 포함된 객체의 내부 구현이 변경되더라도 영향을 최소화할 수 있기 때문에 변경에 더 **안정적인 코드**를 얻을 수 있다.  

상속 관계는 클래스 사이의 정적인 관계, 합성 관계는 동적인 관계이다.  
코드 작성 시점에 결정한 상속 관계는 변경 불가능, 합성 관계는 동적으로 변경 가능  

상속을 합성으로 바꾸는 방법  
자식 클래스에 선언된 상속 관계를 제거하고 부모 클래스의 인스턴스를 자식 클래스의 인스턴스 변수로 선언하면 된다.  

상속대신 합성을 사용하게 되면
1. 불필요한 인터페이스 상속 문제
2. 메서드 오버라이딩의 오작용 문제
3. 부모 클래스와 자식 클래스의 동시 수정 문제

다음과 같은 3가지 문제를 해결할 수 있게 된다.  

---

<h3>상속으로 인한 조합의 폭발적인 증가</h3>

휴대폰 요금의 기본 정책이
1. 일반 요금제
2. 심야 할인 요금제

다음과 같았는데, 부가 정책이 추가로 2가지가 생긴다면 조합 가능한 경우의 수가 굉장히 많아질 것이다.  

```java
public abstract class Phone{
    List<Call> calls;
    
    Money calculateFee();
    abastract protected Money calculateCallFee(Call call);
}

public class RegularPhone extends Phone

public class NightlyDiscountPhone extends Phone
```
기본 정책을 구현한 2개의 클래스이다.  

부가정책 이름을 A,B라고 쳤을 때에 나올 수 있는 클래스는 다음과 같을 것이다.  
```java
public class A_RegularPhone extends RegularPhone
public class B_RegularPhone extends RegularPhone
public class A_B_RegularPhone extends RegularPhone
public class B_A_RegularPhone extends RegularPhone

public class A_NightlyDiscountPhone extends NightlyDiscountPhone
public class B_NightlyDiscountPhone extends NightlyDiscountPhone
public class A_B_NightlyDiscountPhone extends NightlyDiscountPhone
public class B_A_NightlyDiscountPhone extends NightlyDiscountPhone
```
여기서 정책이 1가지라도 더 추가되면 그 수는 또 엄청나게 늘어날 것이다.  

이렇듯, 상속을 남용하게 되면 `클래스 폭발 문제` 또는 `조합의 폭발`문제 라고 불리는 문제가 일어나게 된다.  

---

<h3>합성 관계로 변경하기</h3>

상속은 합성과 달리 런타임에 동적으로 변경할 수 있기에, 합성을 사용하면 **컴파일 타임 의존성과 런타임 의존성을 다르게 만들 수 있다.**  

가장 먼저 해야 할 일은 각 정책을 별도의 클래스로 구현하는 일이다.

```java
//기본 정책과 부가 정책을 포괄하는 RatePolicy 인터페이스
public interface RatePolicy{
    Money calculateFee(Phone phone);
}

//기본정책
public abstract class BasicRatePolicy implements RatePolicy{
    @Override
    public Money calculateFee(Phone phone) {
        Money result = Money.ZERO;
        for (Call call : phone.getCalls()) {
            result.plus(calculateCallFee(call));
        }
        return result;
    }

    protected abstract Money calculateCallFee(Call call);
}

//일반 요금제
public class RegularPolicy extends BasicRatePolicy{
    private Money amount;
    private Duration seconds;

    public RegularPolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

public class Phone{
    private RatePolicy ratePolicy;
    private List<Call> calls = new ArrayList<>();

    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }
    public List<Call> getCalls(){
        return Collections.unmodifiableList(calls);
    }
    public Money calculateFee(){
        return ratePolicy.calculateFee(this);
    }
}
```
Phone 내부에 RatePolicy에 대한 참조자가 포함되어있따. 이것이 바로 합성이다.  
Phone은 이 컴파일 타임 의존성을 구체적인 런타임 의존성으로 대체하기 위해  
생성자를 통해 RatePolicy의 인스턴스에 대한 의존성을 주입받는다.  
![11](https://user-images.githubusercontent.com/45073750/93849726-b1de4400-fce7-11ea-9a60-a91764f751a3.PNG)
다음과 같은 구조를 가지고 있다.  
부가 정책 또한 RatePolicy를 받아서 BasicRatePolicy와 같은 형태로 만들면 된다.  
![12](https://user-images.githubusercontent.com/45073750/93849931-100b2700-fce8-11ea-9fd0-305d84d426eb.PNG)
다음과 같이 말이다.  

그리고 Phone인스턴스 생성 방법은 다음과 같다.  
```java
Phone phone = new Phone(
            new TaxablePolicy(0.05,
                    new RegularPolicy(...));

Phone phone = new Phone(
            new TaxablePolicy(0.05,
                new RateDiscountablePolicy(Money.wons(1000),
                    new RegularPolicy(...))));
```
물론 TaxablePolicy 와 RateDiscountablePolicy의 순서를 바꾼다던지 해도 무리가 없다.  

현재 구조에서 새로운 정책을 추가하고 싶으면 클래스 하나만 추가하고 조합하면 해결이 된다.  
 