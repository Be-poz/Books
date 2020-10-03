일관성 있는 협력
-
객체지향의 목표는 적절한 책임을 수행하는 객체들의 협력을 기반으로 **결합도가 낮고 재사용 가능**한 코드 구조를 창조하는 것이다.  

객체들의 협력 구조가 서로 다른 경우에는 **코드를 이해하기도 어렵고** 코드 수정으로 인해 **버그가 발생할 위험성도 높아진다.**  

협력 방식을 일관성 있게 만들었을 경우의 장점  
1. 객체지향의 특징인 재사용에 들어가는 비용을 감소시킨다.
2. 코드 이해가 쉬워지고 코드의 구조를 예상할 수 있게 된다.  
ex) 새로 접한 코드가 이전에 봤던 코드와 유사하다는 사실을 안 순간 이해하기 편한 경우

  

<h4>**일관성 있는 협력 패턴을 적용하면 코드가 이해하기 쉽고 직관적이고 유연해 진다.**</h4>
***
기존의 핸드폰 요금 정책을 
- 고정요금 방식
- 시간대별 방식
- 요일별 방식
- 구간별 방식

이 4가지로 확장한다.  

구현할 대략적인 구조는 다음과 같다  
![coll1](https://user-images.githubusercontent.com/45073750/93707468-d8767080-fb69-11ea-895c-76779ee6a299.PNG)

  
  
고정요금 방식은 기존의 일반요금제에서 클래스명만 바뀐 형태
```java
public class FixedFeePolicy extends BasicRatePolicy{
    private Money amount;
    private Duration seconds;

    public FixedFeePolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds()/seconds.getSeconds());
    }
}
```

시간대별 방식은 통화의 시작 시간과 종료 시간 뿐만 아니라 시작 일자와 종료 일자도 고려해야 한다.
(통화가 여러 날에 걸쳐서 이루어질 수 있기 때문에)  
시간대별 기간을 편하게 관리할 수 있도록 DateTimeInterval 클래스를 추가한다. 

ToMidnight, FromMidnight 등의 메서드를 넣어 구현한다.  
```java
public class DateTimeInterval{
    private LocalDateTime from;
    private LocalDateTime of;

    public static DateTimeInterval of(LocalDateTime from, LocalDateTime to) {
        return new DateTimeInterval(from, to);
    }

    public static DateTimeInterval toMidnight(LocalDateTime from) {
        return new DateTimeInterval(from,
                LocalDateTime.of(from.toLocalDate(), LocalTime.of(23,59,59)));
    }

    public static DateTimeInterval fromMidnight(LocalDateTime to) {
        return new DateTimeInterval(
                LocalDateTime.of(to.toLocalDate(),LocalTime.of(0,0)),
                to);
    }

    public static DateTimeInterval during(LocalDate date) {
        return new DateTimeInterval(
                LocalDateTime.of(date, LocalTime.of(0, 0)),
                LocalDateTime.of(date,LocalTime.of(23,59,59))
        );
    }

    private DateTimeInterval(LocalDateTime from, LocalDateTime to) {
        this.from = from;
        this.to = to;
    }

    public Duration duration(){
        return Duration.between(from, to);
    }

    public LocalDateTime getFrom() {
        return from;
    }

    public LocalDateTime getTo() {
        return to;
    }

}
```
이전의 Call 클래스는 통화 기간 저장 위한 from, to라는 2개의 LocalDateTime 타입 변수를 가지고 있었는데, 이를 위에서 생성한 DateTimeINteral 타입으로 수정한다.  

```java
public class Call{
    private DateTimeInterval interval;
    
    public Call(LocalDateTime from, LocalDateTime to){
        this.interval = DateTimeInterval.of(from, to);
    }
    
    public Duration getDuration(){
        return interval.duration();
    }
    
    public LocalDateTime getFrom() {
        return interval.getFrom();
    }
    
    public LocalDateTime getTo() {
        return interval.getTo();
    }
    
    public DateTimeInterval getInterval() {
        return interval;
    }
}
```
구현된 코드를 바탕으로  
1. 통화 기간을 일자별로 분리한다.
2. 일자별로 분리된 기간을 다시 시간대별 규칙에 따라 분리한 후 각 기간에 대해 요금을 계산한다.

위와 같은 로직을 거치면 된다.  

시간대별 요금 정보를 알고 있는 TimeOfDayDiscountPolicy를 구현을 하여 Call에게 일자별로 통화 기간 분리를 요청을 하면 Call은 DateTimeInterval에게 위임을 하게 만든다.
```java
public class TimeOfDayDiscountPolicy extends BasicRatePolicy{
    private List<LocalTime> starts = new ArrayList<>();
    private List<LocalTime> ends = new ArrayList<>();
    private List<Duration> durations = new ArrayList<>();
    private List<Money> amounts = new ArrayList<>();

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for (DateTimeInterval interval : call.splitByDay()) {
            for (int loop = 0; loop < starts.size(); loop++) {
                result.plus(amounts.get(loop).times(Duration.between(from(interval, starts.get(loop)), 
                        to(interval, ends.get(loop))).getSeconds() / durations.get(loop).getSeconds));
            }
        }
        return result;
    }

    private LocalTime from(DateTimeInterval interval, LocalTime from) {
        return interval.getFrom().toLocalTime().isBefore(from) ?
                from :
                interval.getFrom().toLocalTime();
    }

    private LocalTime to(DateTimeInterval interval, LocalTime to) {
        return interval.getTo().toLocalTime().isAfter(to) ?
                to :
                interval.getTo().toLocalTime();
    }
}
```



요일별 방식의 경우는 4개의 리스트를 이용한 시간별과는 달리 DayOfWeekDiscountRule이라는 하나의 클래스로 구현하는 것이 더 나은 설계라고 판단해서 새로 구현하였다. 
```java
public class DayOfWeekDiscountRule{
    private List<DatyOfWeek> dayOfWeeks = new ArrayList<>();
    private Duration duration=Duration.ZERO;
    private Money amoun=Money.ZERO;

    public DayOfWeekDiscountRule(List<DayOfWeek> dayOfWeeks, Duration duration, Money amount) {
        this.dayOfWeeks = dayOfWeeks;
        this.duration = duration;
        this. amount = amount;
    }

    public Money calculate(DateTimeInterval interval) {
        if (dayOfWeeks.contains(interval.getFrom().getDayOfWeek())) {
            return amount.times(interval.duration().getSeconds()/duration.getSeconds());
        }
    }
    return Money.ZERO;
}

public class DayOfWeeksDiscountPolicy extends BasicRatePolicy{
    private List<DayOfWeekDiscountRule> rules = new ArrayList<>();

    public DayOfWeekDiscountPolicy(List<DayOfWeeksDiscountRule> rules) {
        this.rules=rules;
    }

    @Override
    protected Money calculateCallFree(Call call) {
        Money result=Money.ZERO;
        for (DateTimeINterval interval : call.getInterval().splitByDay()) {
            for (DayOfWeekDiscountRule rule : rules) {
                result.plus(rule.calculate(interval));
            }
        }
        return result;
    }
}
```  
이제 구간별 방식을 구현을 해야하는데, 앞서 구현한 2가지 방식은 개별로 보면 괜찮은 구현이지만 문제점이 존재하는데 다음과 같다.
1. 새로운 구현을 추가해야 하는 상황
2. 기존의 구현을 이해해야 하는 상황  

고정요금 방식까지 합하면 세 가지 기본 정책에 대해서 세 가지 서로 다른 구현 방식이 존재한다는 것을 알 수 있다.  
**전체적인 코드 사이의 일관성이 어긋나기 시작했고, 한 가지 정책에 대한 구현방법을 이해해도 다른 정책의 구현을 이해하는데에 도움을 주지 않는다.**  

***
설계에는 일관성이 필요하다. 일관성 있게 만들기 위한 기본 지침은 다음과 같다.
1. 변하는 개념을 변하지 않는 개념으로부터 분리하라.
2. 변하는 개념을 캡슐화하라.  

4장에서 구현한 ReservationAgency의 기본 구조를 살펴보면  
```java
public class ReservationAgency{
    public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
        for (DiscountCondition condition : movie.getDiscountConditions()) {
            if (conditions.getType() == DiscountConditionType.PERIOD) {
                //기간 조건인 경우
            } else {
                //회차 조건인 경우
            }
        }
        if (discountable) {
            switch (movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    //금액 할인 정책인 경우
                case PERCENT_DISCOUNT:
                    //비율 할인 정책인 경우
                case NONE_DISCOUNT:
                    //할인 정책이 없는 경우
            }
        } else {
            //할인 적용이 불가능한 경우
        }
    }
}
```
 다음과 같은데 이 설계는 변경의 주기가 서로 다른 코드가 한 클래스 안에 뭉쳐있기 때문에 나쁜 설계이다.  
새로운 할인 정책/조건을 추가하기 위해서는 기존 코드의 내부를 수정해야 하기 때문에 오류 발생 확률이 높아진다.  

**객체지향에서 변경을 다루는 전통적인 방법은 조건 로직을 객체 사이의 이동으로 바꾸는 것이다.**  

위의 코드를 다음과 같이 수정해보면
```java
public class Movie{
    private DiscountPolicy discountPolicy;

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAMount(screening));
    }
}

public abstract class DiscountPolicy{
    private List<DiscountCondition> conditions = new ArrayList<>();

    public Money calculateDiscountAmount(Screening screening) {
        for (DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAMount(screening);
            }
        }
        return screening.getMovieFee();
    }
}
```
Movie 클래스에서 현재의 할인 정책이 어떤 종류인지 확인하지 않고, discountPolicy한테 메세지를 전송만 해주고 있다.  
다음과 같은 계층을 보이고 있다.  
![col2](https://user-images.githubusercontent.com/45073750/93710198-bb4c9c80-fb7f-11ea-9060-d22403e2876d.PNG)

큰 메서드 안에 뭉쳐있던 조건 로직들을 변경의 압력에 맞춰 작은 클래스들로 분리하고 나면 인스턴스들 사이의 협력 패턴에 일관성을 부여하기가 더 쉬워진다.  

이러한 설계를 이 장의 첫 예시로 나왔던 할인 정책에도 적용할 수가 있다.  
***

고정요금 방식, 시간대별 방식, 요일별 방식, 구간별 방식의 공통점은 다음과 같다.  
- 한 개 이상의 '규칙'으로 구성된다.
- 하나의 '규칙'은 '적용조건'과 '단위요금'의 조합이다.

차이점은 '적용조건'의 형식이 다르다는 점이다.  
시간대별 방식은 '00시~19시 까지'라는 조건이라면, 구간별 방식은 '초기 1분동안' 와 같이 말이다.  

변하지 않는 것은 '규칙'이고, 변하는 것은 '적용조건'이다.  
'규칙'으로부터 '적용조건'을 분리해서 추상화한 후 시간대별, 요일별, 구간별 방식을 추상화의 서브타입으로 만들면 된다.  
이것을 **서브타입 캡슐화** 라고 부른다.  
![col3](https://user-images.githubusercontent.com/45073750/93710486-31ea9980-fb82-11ea-933d-311bef54b912.PNG)
다음과 같은 다이어그램을 가지게 될 것이다.  

- FeeRule은 '규칙'을 구현하는 클래스이며 '단위요금'은 FeeRule의 인스턴스 변수인 feePerdutaion에 저장되어있다.  
- FeeCondition은 '적용조건'을 구현하는 인터페이스이며 변하는 부분을 캡슐화하는 추상화다.  

![col4](https://user-images.githubusercontent.com/45073750/93710715-ad007f80-fb83-11ea-9e62-a0d6a0f845a1.PNG)
- FeeRule은 FeeCondition의 인스턴스에게 findTimeIntervals 메세지를 전송한다.
- findTimeIntervals는 '적용조건'을 만족하는 구간을 가지는 DateTimeInterval의 list를 반환한다.
- FeeRule은 feePerduration 정보를 이용해 반환받은 기간만큼의 통화 요금을 계산하여 반환한다.

이것을 구현해보면 다음과 같이 구현할 수 있다.  
```java
public interface FeeCondition{
    List<DateTimeInterval> findTimeIntervals(Call call);
}
```
'적용조건'을 표현하는 FeeCondition의 findTimeIntervals는 인자로 전달된 Call의 통화 기간 중에서 '적용조건'을 만족하는 기간을 구해 list에 담는다.  

```java
public class FeeRule{
    private FeeCondition feeCondition;
    private FeePerDuration feePerDuration;

    public FeeRule(FeeCondition feeCondition, FeePerDuration feePerDuration) {
        this.feeCondition = feeCondition;
        this.feePerDuration = feePerDuration;
    }

    public Money calculateFee(Call call) {
        return feeCondition.findTimeIntervals(call)
                .stream()
                .map(each -> feePerDuration.calculate(each))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }
}
```
FeeRule은 '단위요금(feePerDuration)'과 '적용조건(feeCondition)'으로 구성되어 있다.  
calculateFee 메서드에서 FeeCondition의 findTimeIntervals를 통해 조건에 만족하는 시간목록을 받아 feePerDuration 값을 이용해 요금을 계산한다.  
```java
public class FeePerDuration{
    private Money fee;
    private Duration duration;

    public FeePerDuration(Money fee, Duration duration) {
        this.fee = fee;
        this.duration = duration;
    }

    public Money calculate(DateTimeInterval interval) {
        return fee.times(interval.duration().getSeconds() / duration.getSeconds());
    }
}
```
FeePerDuration 클래스에서는 요금을 계산하는 역할을 맡는다.  
```java
public class BasicRatePolicy implements RatePolicy{
    private List<FeeRule> feeRules = new ArrayList<>();

    public BasicRatePolicy(FeeRule... feeRules) {
        this.feeRules = Arrays.asList(feeRules);
    }

    @Override
    public Money calculateFee(Phone phone) {
        return phone.getCalls()
                .stream()
                .map(call -> callculate(call))
                .reduce(Money.ZERO, (first, seond) -> first.plus(second));
    }

    private Money calculate(Call call) {
        return feeRules
                .stream()
                .map(rule -> rule.calculateFee(call))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }
}
```
BasicRatePolicy 까지 구현함으로서 추상적인 수준에서의 협력이 완료됐다.  
시간대별 요금 방법을 예시로 구현을 해보면 다음과 같다.
```java
public class TimeOfDayFeeCondition implements FeeCondition {
    private LocalTime from;
    private LocalTime to;

    public TimeOfDayFeeCondition(LocalTime form, LocalTime to) {
        this.from = from;
        this.to = to;
    }

    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return call.getInterval().splitByDay()
                .stream()
                .map(each->DateTImeInterval.of(
                        LocalDateTime.of(each.getFrom().toLocalDate(),from(each)),
                        LocalDateTime.of(each.getTo().toLocalDate(),to(each))
                )).collect(Collectors.toList());
    }

    private LocalTime from(DateTimeInterval interval) {
        return interval.getFrom().toLocalTime().isBefore(from)?
                from:interval.getFrom().toLocalTime();
    }

    private LocalTime to(DateTimeInterval interval) {
        return interval.getTo().toLocalTime().isAfter(to)?
                to:inteval.getTo().toLocalTime();
    }
}
```
FeeCondition을 implement하여 구현하였고, 나머지 정책들도 이와같이 구현을 하게된다.

- 변하는 부분을 변하지 않는 부분으로부터 분리했기 때문에 **변하지 않는 부분을 재사용**할 수 있다.
- 새로운 기능을 추가하기 위해 오직 변하는 부분만 구현하면 되기 때문에 원하는 기능을 쉽게 완성할 수 있다.
- 코드의 재사용성이 향상되고 테스트해야 하는 코드의 양이 감소한다.
- 기능을 추가 변경해도 설계의 일관성이 무너지지 않는다.
- 기본 정책을 추가하고 싶을 때에도 FeeConditino을 구현하고 FeeRule과 연결하면 끝이다.

최종적인 설계의 클래스 다이어그램은 다음과 같이 마무리 된다.
![col5](https://user-images.githubusercontent.com/45073750/93711436-44b49c80-fb89-11ea-9c9a-769aeef3e9f3.PNG)
***
