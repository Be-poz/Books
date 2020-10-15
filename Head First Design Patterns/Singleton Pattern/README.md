본 내용은 Head First Design Patterns 책을 토대로 정리하였습니다.

> A: 어떤 용도로 사용하나요?  
>   
> B: 객체 중에는 하나만 있으면 되는 것이 많습니다. 스레드 풀, 캐시, 레지스트리 설정 등등... 이런 형식의 객체를 쓸 때에 인스턴스를 두 개 이상 만들게 되면 프로그램이 이상하게 돌아간다든가 자원을 불필요하게 잡아먹는다든가 결과에 일관성이 없어지는 심각한 문제를 초래할 수 있습니다.  
>   
> A: 그러면 그냥 정적 변수 같은걸 쓰면 되지 않나요?  
>   
> B: 싱글턴 패턴은 전역 변수를 사용할 때와 마찬가지로 객체 인스턴스를 어디서든지 엑세스할 수 있게 만들었습니다. 전역 변수에 만약 객체를 대입하면 애플리케이션이 시작될 때 객체가 생성될 것인데, 그 객체가 자원을 많이 차지한다고 가정하고 애플리케이션이 끝날 때까지 그 객체를 한 번도 쓰지 않는다면 자원만 잡아먹는 객체가 되겠죠? 이런 단점 없이 싱글턴 패턴을 쓰면 필요할 때만 객체를 만들 수가 있습니다.  
>   
> A: 객체를 어떻게 생성하죠?  
>   
> B: new MyObject();  
>   
> A: 다른 객체에서 MyObject를 만들고 싶어한다면 다시 new 연산자를 쓸 수 있나요?   
>   
> B: Yes  
>   
> A: 클래스만 있으면 언제든지 인스턴스를 만들 수가 있죠?  
>   
> B: public으로 선언된 거라면 문제 없습니다.  
>   
> A: 그렇다면   
>   public MyClass{  
>     private MyClass() {}  
>   }                              이거는요?  
>   
> B: 문법적으로는 전혀 문제가 없지만 생성자가 private 이기 때문에 인스턴스를 만들 수 없는 클래스입니다. 즉 객체의 인스턴스를 만들 수가 없겠네요  
>   
> A: public MyClass{  
>       public static MyClass getInstance(){}  
>    }                                                      이거는요?  
>   
> B: getInstance가 정적 메서드네요 정적 메서드는 Myclass.getInstance() 와 같은 식으로 호출할 수 있습니다. 정적 메서드는  
>    클래스 메서드라고도 부르는데, 정적 메서드를 지칭할 때에는 클래스 이름을 써야 합니다.  
>   
> A: 그렇다면 public static MyClass getInstance(){  
>                      return new MyClass(); }             이것이라면 인스턴스를 만들 수 있겠네요??  
>   
> B: 그렇습니다. MyClass.getInstance(); 를 사용하면 인스턴스가 만들어 지겠네요.

그렇다. 싱글턴 패턴은 하나만 있어야 하는 객체를 위한 패턴이다. 

private을 생성자의 접근자로 달아주면서 생성을 못하게 만들고 정적 메서드를 통해서 인스턴스를 생성하는 것이다.

그렇다면 정확히 어떤식으로 구현해야 할까?

```java
public class Singleton{
    private static Singleton uniqueInstance;
    
    private Singleton() {}
 
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
           uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

다음과 같이 uniqueInstance가 null 값이라면 생성자를 통해 만들고 이미 존재한다면 그것을 return 해준다.

그런데 위의 코드로 사용하던 중 2개의 인스턴스가 생겼고 큰 문제가 생겼다.

왜일까???

그 이유는 바로 멀티스레드를 사용했을 때에 위의 코드는 완전하지 않기 때문이다.

                 1번 스레드                                                     2번 스레드

public static Singleton getInstance(){

                                                                    public static Singleton getInstance(){

if(uniqueInstance==null){

                                                                   if(uniqueInstance==null){

uniqueInstance = new Singleton();

return uniqueInstance;

                                                                    uniqueInstance=new Singleton();

                                                                    return uniqueInstance;

이해가 됐나요? 

이런 상황때문에 그렇습니다.

멀티스레딩 문제를 해결하기 위해서는 getInstance() 메서드를 동기화 시켜주면 됩니다.

```java
    public static synchronized Singleton getInstance() {
        if (uniqueInstance == null) {
           uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
```

다음과 같이 말입니다. 하지만 이 경우에는 속도 문제가 생길 것입니다.

이 문제를 어떻게 해결할까요?

1\. getInstance() 메서드가 애플리케이션에 큰 부담을 주지 않는다면 그냥 내버려둡니다.

2\. 인스턴스를 필요할 때 생성하지 말고, 처음부터 만들어 버립니다.

```java
public class Singleton{
    private static Singleton uniqueInstance = new Singleton();
 
    private Singleton() {}
 
    public static synchronized Singleton getInstance() {
 
        return uniqueInstance;
    }
}
```

3\. DCL(Double-Checking-Locking) 을 써서 getInstance()에서 동기화되는 부분을 줄입니다.

```java
public class Singleton{
    private volatile static Singleton uniqueInstance;
 
    private Singleton() {}
 
    public static synchronized Singleton getInstance() {
        if(uniqueInstance == null) {
            synchronized (Singleton.class) {
                if(uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

인스턴스가 있는지 확인하고 없으면 동기화된 블럭으로 들어갑니다. 이렇게 처리하면 처음에만 동기화가 됩니다.

블럭으로 들어간 후에도 다시 한 번 변수가 null 인지 확인한 다음 인스턴스를 생성했습니다.

volatile 키워드를 사용해서 멀티스레딩을 쓰더라도 uniqueInstance 변수가 Singleton 인스턴스로 초기화 되는 과정이 올바르게 진행되도록 할 수 있다. 상세 이유는 밑 포스팅을 참조

[자바에서 volatile 이란?](https://bepoz-study-diary.tistory.com/160)   

***
추가적으로 보충하자면,  
싱글턴 패턴을 사용할 때 주의해야 될점은 다음과 같다.  
1. 싱글턴은 무상태(stateless)로 설계해야 한다.  
2. 특정 클라이언트에 의존적인 필드가 있으면 안된다.  
3. 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.  
4. 가급적 읽기만 가능해야 한다.  
5. 필드 대신에 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다. 



싱글턴 패턴의 단점은 다음과 같다.

1. 싱글턴 패턴을 구현하는 코드 자체가 많이 들어간다.
2. 의존관계상 클라이언트가 구체 클래스에 의존한다. -> DIP를 위반한다.
3. 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
4. 테스트하기 어렵다.
5. 내부 속성을 변경하거나 초기화 하기 어렵다.
6. private 생성자로 자식 클래스를 만들기 어렵다.
7. 결론적으로 유연성이 떨어진다.
8. 안티패턴으로 불리기도 한다.