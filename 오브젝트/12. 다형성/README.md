다형성
-

<h3> 다형성 </h3>

다형성의 분류는 다음과 같이 나눌 수 있다.
* 다형성
  * 유니버설
    * 매개변수
    * 포함
  * 임시
    * 오버로딩
    * 강제
    
<h4>오버로딩 다형성</h4>
```java
public class Money{
    public Money plus(Money amount){...}
    public Money plus(BigDecimal amount){...}
    public Money plus(long amount){...}
}
```
위와 같은 경우를 오버로딩 다형성이라고 부른다.



<h4>강제 다형성</h4>
* 언어가 지원하는 자동적인 타입 변환이나 사용자가 직접 구현한 타입 변환을 이용해 동일한 연산자를 다양한 타입에 사용할 수 있는 방식  
ex) '+' 의 경우 정수와 정수사이에서는 덧셈이지만, 정수와 문자열일 경우 연결 연산자로 작동하는 것과 같은 것  
   이때, 정수형 피연산자는 문자열 타입으로 강제 형변환된다. 오버로딩 다형성과 함께 사용하면 모호해질 수가 있다.



<h4>매개변수 다형성</h4>
* 매개변수 타입을 임의의 타입으로 선언한 후 사용하는 시점에 구체적인 타입으로 지정하는 방식  
ex) List 인터페이스에 요소의 타입을 임의의 타입인 T로 지정한 것 과 같은 것  



<h4>포함 다형성</h4>
* 메세지가 동일하더라도 수신한 객체의 타입에 따라 실제로 수행되는 행동이 달라지는 능력을 의미. `서브타입 다형성`이라고도 부른다.  
```java
public class Movie{
    private DiscountPolicy discountPolicy;

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```
위의 경우가 예시인데, calculateDiscountAmount는 수신받은 메세지에 따라 실행되는 메서드가 달라진다.  

* 두 클래스를 상속 관계로 연결하고 자식 클래스에서 부모 클래스의 메서드를 오버라이딩한 후 클라이언트가 부모 클래스만 참조하면 포함 다형성을 구현할 수 있다.  

* 포함 다형성을 위한 전제조건은 자식 클래스가 부모 클래스의 서브타입이어야 한다는 것이다.  
(상속의 진정한 목적은 코드 재사용이 아니라 다형성을 위한 서브타입 계층을 구축하는 것이다)

***

<h3> 상속의 양면성 </h3>

상속의 목적은 코드 재사용이 아닌, 프로그램을 구성하는 개념들을 기반으로 다형성을 가능하게 하는 타입 계층을 구축하기 위한 것이다.  

상속의 매커니즘을 이해하는 데 필요한 개념은 다음과 같다.
* 업캐스팅
* 동적 메서드 탐색
* 동적 바인딩
* self 참조
* super 참조

이번 장에서 쓰일 코드를 먼저 보면
```java
public class Lecture{
    private int pass;
    private String title;
    private List<Integer> scores = new ArrayList<>();

    public Lecture(String title, int pass, List<Integer> scores) {
        this.title=title;
        this.pass=pass;
        this.scores=scores;
    }
    public double average(){
        return scores.stream()
                .ampToInt(Integer::intValue)
                .average().orElse(0);
    }
    public List<Integer> getScores{
        return Collections.unmodifiableList(scores);
    }
    public String evaluate(){
        return String.format("Pass:%d Fail:%d", passCount(), failCount());
    }
    public long passCount(){
        return scores.stream().filter(score->score>=pass).count();
    }
    private long failCount() {
        return scores.size() - passCount();
    }
}

public class Grade{
    private String name;
    private int upper, lower;

    private Grade(String name, int upper, int lower) {
        this.name=name;
        this.upper=upper;
        this.lower=lower;
    }
    public String getName(){
        return name;
    }

    public boolean isName(String name) {
        return this.name.equals(name);
    }

    public boolean include(int score) {
        return score>=lower&&score<=upper;
    }
}

public class GradeLecture extends Lecture {
    private List<Grade> grades;

    public GradeLEcture(String name, int pass, List<Grade> grades, List<Integer> scores) {
        super(name, pass, scores);
        this.grades = grades;
    }

    @Override
    public String evaluate(){
        return super.evaluate()+", "+gradesStatistics();
    }
    private String gradesStatistics(){
        return grades.stream()
                .map(grade->format(grade))
                .collect(joining(" "));
    }

    private String format(Grade grade) {
        return String.format("%s:%d", grade.getName(), gradeCount(grade));
    }
    private long gradeCoung(Grade gradereturn getScores().stream()
                .filter(grade::include).count();
    )
}
```
* GradeLecture, Lecture의 evaluate 메서드는 시그니처가 완전히 동일한데, 자식 클래스의 우선순위가 부모 클래스보다 높다. 이처럼 자식 클래스 안에 상속받은 메서드와 동일한 시그니처의
메서드를 재정의해서 부모 클래스의 구현을 새로운 구현으로 대체하는 것을 `메서드 오버라이딩`
* GradeLecture, Lecture의 average는 이름은 같지만 시그너치가 다르다. 클라이언트는 두 메서드 모두를 호출할 수 있다.  
이처럼 부모 클래스에서 정의한 메서드와 이름은 동일하지만 시그니처는 다른 메서드를 자식 클래스에 추가하는 것을 `메서드 오버로딩`

구조를 다음과 같이 생각할 수가 있다.  
![12-1](https://user-images.githubusercontent.com/45073750/93854874-7fd1df80-fcf1-11ea-80ff-828a18ff3235.PNG)
그리고 또한 다음과 같이 생각해도 무방할 것이다.
![1202](https://user-images.githubusercontent.com/45073750/93854879-82343980-fcf1-11ea-9282-3dafaa80c63c.PNG)
객체의 경우에는 서로 다른 상태를 저장할 수 있도록 각 인스턴스 별로 독립적인 메모리를 할당받아야 하지만, 메서드의 경우에는 동일한 클래스의 인스턴스끼리 공유가 가능하기 때문에
클래스는 한 번만 메모리에 로드하고 각 인스턴스별로 클래스를 가리키는 포인터를 갖게 하는 것이 경제적이다.
![12-3](https://user-images.githubusercontent.com/45073750/93854884-83656680-fcf1-11ea-96b6-1d6790b68e58.PNG)
위의 사진은 인스턴스가 2개 생성되었지만 클래스는 단 하나만 메모리에 로드됐다는 것에 유의하면 된다.
![12-4](https://user-images.githubusercontent.com/45073750/93854886-84969380-fcf1-11ea-90df-dac2a752cd15.PNG)
GradeLecture의 전체적인 구조는 다음과 같다고 볼 수가 있다.  

***

<h3> 업캐스팅과 동적 바인딩 </h3>

```java
public class Professor{
    private String name;
    private Lecture lecture;

    public Professor(String name, Lecture lecture) {
        this.name=name;
        this.lecture=lecture;
    }
    
    public String compileStatistics(){
        return String.format("[%s] %s - Avg: %.1f", name, lecture.evaluate(), lecture.average());
    }
}

Professore professor = new Professor("다익스트라",
        new Lecture("알고리즘",
                70, Arrays.asList(81, 95, 75, 50, 45)));
//결과 => "[다익스트라] Pass:3 Fail:2 - Avg: 69.2
//String statistics = professor.compileStatistics();
```
위 코드에서 Lecture 대신 GradeLecture를 사용해도 실행이 되었을 것이다.  
* 부모 클래스(Lecture) 타입으로 선언된 변수에 자식 클래스(GradeLecture)의 인스턴스를 할당하는 것이 가능하고 이를 `업캐스팅`이라고 부를ㄴ다.
* 선언된 변수의 타입이 아니라 메세지를 수신하는 객체의 타입에 따라 실행되는 메서드가 결정된다.
이것은 객체지향 시스템이 메세지를 처리할 적절한 메서드를 컴파일 시점이 아니라 실행 시점에 결정하기 때문에 가능하다. 이를 `동적바인딩`이라고 부른다.

```java
Lecture lecture = new GradeLecture(...);
GradeLecture gradeLecture = (GradeLecture)lecture;
```
반대로 다음과 같이 부모클래스의 인스턴스를 자식 클래스 타입으로 변환하기 위해서는 명시적인 타입 캐스팅이 필요한데 이를 `다운캐스팅`이라고 부른다.  

함수를 호출하는 전통적인 언어들은 호출될 함수를 컴파일 타임에 결정한다. bar함수 호출 구문이 나온다면 실제로 실행되는 코드는 바로 그 bar 함수다. 코드를 작성하는 시점에 호출될 코드가 결정된다.  
이처럼 컴파일타임에 호출할 함수를 결정하는 방식을 `정적 바인딩`, `초기 바인딩`, `컴파일타임 바인딩`이라고 부른다.  

객체지향 언어에서는 메세지를 수신했을 때 실행될 메서드가 런타임에 결정된다. 실행되는 함수가 어떤 클래스의 어떤 메서드인지 판단하기가 어렵다.
foo.bar()라는 코드에서 foo가 가리키는 객체가 실제로 어떤 클래스의 인스턴스인지를 알아야 하고 bar 메서드가 해당 클래스의 어디에 위치하는지를 알아야한다.  
이처럼 실행될 메서드를 런타임에 결정하는 방식을 `동적 바인딩` 또는 `지연 바인딩` 이라고 부른다.

***

<h3> 동적 메서드 탐색과 다형성 </h3>

객체지향 시스템은 다음 규칙에 따라 실행할 메서드를 선택한다.
* 자신을 생성한 클래스에 적합한 메서드가 존재하는지 검사한다. 존재하면 메서드를 실행하고 탐색을 종료한다.
* 찾지 못했다면 부모 클래스에서 탐색을 계속한다. 이 과정은 메서드를 찾을 때까지 상속 계층을 올라간다.
* 상속 계층의 가장 최상위 클래스에 이르렀지만 메서드를 발견하지 못한 경우 예외를 발생시키며 탐색을 종료한다.

이 과정에서 **self 참조**를 이용해서 탐색을 하게된다.  
수신한 객체에서 시작해 계층을 올라가며 메서드를 탐색한다.  

동적 메서드 탐색은 2가지 원리로 구성됨을 알 수 있다.
1. **자동적인 메세지 위임**  
자식 클래스는 자신이 이해할 수 없는 메세지를 받은 경우 부모 클래스에게 처리를 위임한다.
2. **동적인 문맥**  
메세지 수신 시 메서드 실행의 결정 시점은 컴파일이 아닌 실행 시점이고, 메서드 탐색 경로는 self 참조를 이용한다.

오버라이딩된 메서드를 실행하는 과정도 이 self 참조를 통해서 계층을 올라가면서 탐색을 하는데,  
자식 클래스에서 발견했으면 바로 탐색이 종료되기 때문에 자식의 메서드부터 참조되는 것이다. 

메서드 오버로딩 또한 동일한 방법으로 탐색을 한다. C++은 상속 계층 사이의 메서드 오버로딩을 지원하지 않는다.  

동적인 문맥을 결정하는 것은 self 참조이다. self 참조에 대한 재밌는 사실을 한 번 알아보겠다.  
```java
public class Lecture{
    public String stats(){
        return String.format("Title: %s Evaluation Method: %s",title, getEvaluationMethod());
    }
    
    public String getEvalutationMethod(){
        return "PASS or FAIL";
    }
}
```
Lecture에서 stats() 메서드 내에 getEvaluationMethod()를 호출한다고 보통 표현하지만 사실 이 말은 정확하지 않다.  
getEvaluationMethod()라는 구문은 현재 클래스의 메서드를 호출하는 것이 아니라 현재 객체에게 말하는 것이다.  

현재 객체는 self 참조가 가리키는 객체를 말한다.  
![12-5](https://user-images.githubusercontent.com/45073750/93860179-a4ca5080-fcf9-11ea-96ef-54cf963212fa.PNG)
위의 상황에서 stats() 메서드를 만나고 그 내부에서 getEvaluationMethod()를 호출하는데, 메서드 탐색은 처음에 메세지 탐색을 시작했던 self 참조가 가리키는 그 클래스에서부터 다시 시작하게 된다.  
![12-6](https://user-images.githubusercontent.com/45073750/93860184-a6941400-fcf9-11ea-9ccb-41d71e40ca31.PNG)
따라서, Lecture를 받은 GradeLecture에서 getEvaluationMethod()를 오버라이딩 했다면 Lecture의 stats() 메서드 내에서 호출된 getEvaluationMethod()가
self 참조부터 탐색을 다시 시작하면서 GradeLecture의 getEvaluationMethod()를 호출하게 된다는 것이다.  

super의 경우는 부모클래스부터 시작하는 self 참조라고 생각하면 이해하기 쉽다.  
부모 클래스에서부터 탐색을 시작하는 것이다.  

***