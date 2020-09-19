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

<table class="colorscripter-code-table" style="margin: 0; padding: 0; border: none; background-color: #fafafa; border-radius: 4px;" cellspacing="0" cellpadding="0"><tbody><tr><td style="padding: 6px; border-right: 2px solid #e5e5e5;"><div style="margin: 0; padding: 0; word-break: normal; text-align: right; color: #666; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace !important; line-height: 130%;"><div style="line-height: 130%;">1</div><div style="line-height: 130%;">2</div><div style="line-height: 130%;">3</div><div style="line-height: 130%;">4</div><div style="line-height: 130%;">5</div><div style="line-height: 130%;">6</div><div style="line-height: 130%;">7</div><div style="line-height: 130%;">8</div><div style="line-height: 130%;">9</div><div style="line-height: 130%;">10</div><div style="line-height: 130%;">11</div><div style="line-height: 130%;">12</div><div style="line-height: 130%;">13</div></div></td><td style="padding: 6px 0; text-align: left;"><div style="margin: 0; padding: 0; color: #010101; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace !important; line-height: 130%;"><div style="padding: 0 6px; white-space: pre; line-height: 130%;"><span style="color: #ff3399;">public</span>&nbsp;<span style="color: #ff3399;">class</span>&nbsp;Singleton{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">private</span>&nbsp;<span style="color: #ff3399;">static</span> Singleton uniqueInstance;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">private</span>&nbsp;Singleton()&nbsp;{}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">public</span>&nbsp;<span style="color: #ff3399;">static</span>&nbsp;Singleton&nbsp;getInstance()&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">if</span> (uniqueInstance <span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span><span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span>&nbsp;<span style="color: #0099cc;">null</span>)&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;uniqueInstance <span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span>&nbsp;<span style="color: #ff3399;">new</span>&nbsp;Singleton();</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">return</span> uniqueInstance;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;</div></div><div style="text-align: right; margin-top: -13px; margin-right: 5px; font-size: 9px; font-style: italic;"><a style="color: #e5e5e5text-decoration:none;" href="http://colorscripter.com/info#e" target="_blank" rel="noopener">Colored by Color Scripter</a></div></td><td style="vertical-align: bottom; padding: 0 2px 4px 0;"><a style="text-decoration: none; color: white;" href="http://colorscripter.com/info#e" target="_blank" rel="noopener"><span style="font-size: 9px; word-break: normal; background-color: #e5e5e5; color: white; border-radius: 10px; padding: 1px;">cs</span></a></td></tr></tbody></table>

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

<table class="colorscripter-code-table" style="margin: 0; padding: 0; border: none; background-color: #fafafa; border-radius: 4px;" cellspacing="0" cellpadding="0"><tbody><tr><td style="padding: 6px; border-right: 2px solid #e5e5e5;"><div style="margin: 0; padding: 0; word-break: normal; text-align: right; color: #666; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace !important; line-height: 130%;"><div style="line-height: 130%;">1</div><div style="line-height: 130%;">2</div><div style="line-height: 130%;">3</div><div style="line-height: 130%;">4</div><div style="line-height: 130%;">5</div><div style="line-height: 130%;">6</div></div></td><td style="padding: 6px 0; text-align: left;"><div style="margin: 0; padding: 0; color: #010101; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace !important; line-height: 130%;"><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">public</span>&nbsp;<span style="color: #ff3399;">static</span>&nbsp;<span style="color: #ff3399;">synchronized</span>&nbsp;Singleton&nbsp;getInstance()&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">if</span> (uniqueInstance <span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span><span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span>&nbsp;<span style="color: #0099cc;">null</span>)&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;uniqueInstance <span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span>&nbsp;<span style="color: #ff3399;">new</span>&nbsp;Singleton();</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">return</span> uniqueInstance;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;}</div></div><div style="text-align: right; margin-top: -13px; margin-right: 5px; font-size: 9px; font-style: italic;"><a style="color: #e5e5e5text-decoration:none;" href="http://colorscripter.com/info#e" target="_blank" rel="noopener">Colored by Color Scripter</a></div></td><td style="vertical-align: bottom; padding: 0 2px 4px 0;"><a style="text-decoration: none; color: white;" href="http://colorscripter.com/info#e" target="_blank" rel="noopener"><span style="font-size: 9px; word-break: normal; background-color: #e5e5e5; color: white; border-radius: 10px; padding: 1px;">cs</span></a></td></tr></tbody></table>

다음과 같이 말입니다. 하지만 이 경우에는 속도 문제가 생길 것입니다.

이 문제를 어떻게 해결할까요?

1\. getInstance() 메서드가 애플리케이션에 큰 부담을 주지 않는다면 그냥 내버려둡니다.

2\. 인스턴스를 필요할 때 생성하지 말고, 처음부터 만들어 버립니다.

<table class="colorscripter-code-table" style="margin: 0; padding: 0; border: none; background-color: #fafafa; border-radius: 4px;" cellspacing="0" cellpadding="0"><tbody><tr><td style="padding: 6px; border-right: 2px solid #e5e5e5;"><div style="margin: 0; padding: 0; word-break: normal; text-align: right; color: #666; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace !important; line-height: 130%;"><div style="line-height: 130%;">1</div><div style="line-height: 130%;">2</div><div style="line-height: 130%;">3</div><div style="line-height: 130%;">4</div><div style="line-height: 130%;">5</div><div style="line-height: 130%;">6</div><div style="line-height: 130%;">7</div><div style="line-height: 130%;">8</div><div style="line-height: 130%;">9</div><div style="line-height: 130%;">10</div></div></td><td style="padding: 6px 0; text-align: left;"><div style="margin: 0; padding: 0; color: #010101; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace !important; line-height: 130%;"><div style="padding: 0 6px; white-space: pre; line-height: 130%;"><span style="color: #ff3399;">public</span>&nbsp;<span style="color: #ff3399;">class</span>&nbsp;Singleton{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">private</span>&nbsp;<span style="color: #ff3399;">static</span>&nbsp;Singleton&nbsp;uniqueInstance&nbsp;<span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span>&nbsp;<span style="color: #ff3399;">new</span>&nbsp;Singleton();</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">private</span>&nbsp;Singleton()&nbsp;{}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">public</span>&nbsp;<span style="color: #ff3399;">static</span>&nbsp;<span style="color: #ff3399;">synchronized</span>&nbsp;Singleton&nbsp;getInstance()&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">return</span>&nbsp;uniqueInstance;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">}</div></div><div style="text-align: right; margin-top: -13px; margin-right: 5px; font-size: 9px; font-style: italic;"><a style="color: #e5e5e5text-decoration:none;" href="http://colorscripter.com/info#e" target="_blank" rel="noopener">Colored by Color Scripter</a></div></td><td style="vertical-align: bottom; padding: 0 2px 4px 0;"><a style="text-decoration: none; color: white;" href="http://colorscripter.com/info#e" target="_blank" rel="noopener"><span style="font-size: 9px; word-break: normal; background-color: #e5e5e5; color: white; border-radius: 10px; padding: 1px;">cs</span></a></td></tr></tbody></table>

3\. DCL(Double-Checking-Locking) 을 써서 getInstance()에서 동기화되는 부분을 줄입니다.

<table class="colorscripter-code-table" style="margin: 0; padding: 0; border: none; background-color: #fafafa; border-radius: 4px;" cellspacing="0" cellpadding="0"><tbody><tr><td style="padding: 6px; border-right: 2px solid #e5e5e5;"><div style="margin: 0; padding: 0; word-break: normal; text-align: right; color: #666; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace !important; line-height: 130%;"><div style="line-height: 130%;">1</div><div style="line-height: 130%;">2</div><div style="line-height: 130%;">3</div><div style="line-height: 130%;">4</div><div style="line-height: 130%;">5</div><div style="line-height: 130%;">6</div><div style="line-height: 130%;">7</div><div style="line-height: 130%;">8</div><div style="line-height: 130%;">9</div><div style="line-height: 130%;">10</div><div style="line-height: 130%;">11</div><div style="line-height: 130%;">12</div><div style="line-height: 130%;">13</div><div style="line-height: 130%;">14</div><div style="line-height: 130%;">15</div><div style="line-height: 130%;">16</div></div></td><td style="padding: 6px 0; text-align: left;"><div style="margin: 0; padding: 0; color: #010101; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace !important; line-height: 130%;"><div style="padding: 0 6px; white-space: pre; line-height: 130%;"><span style="color: #ff3399;">public</span>&nbsp;<span style="color: #ff3399;">class</span>&nbsp;Singleton{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">private</span>&nbsp;<span style="color: #ff3399;">volatile</span>&nbsp;<span style="color: #ff3399;">static</span>&nbsp;Singleton&nbsp;uniqueInstance;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">private</span>&nbsp;Singleton()&nbsp;{}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">public</span>&nbsp;<span style="color: #ff3399;">static</span>&nbsp;<span style="color: #ff3399;">synchronized</span>&nbsp;Singleton&nbsp;getInstance()&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">if</span>(uniqueInstance&nbsp;<span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span><span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span>&nbsp;<span style="color: #0099cc;">null</span>)&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">synchronized</span>&nbsp;(Singleton.<span style="color: #ff3399;">class</span>)&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">if</span>(uniqueInstance&nbsp;<span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span><span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span>&nbsp;<span style="color: #0099cc;">null</span>)&nbsp;{</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;uniqueInstance&nbsp;<span style="color: #0086b3;"></span><span style="color: #ff3399;">=</span>&nbsp;<span style="color: #ff3399;">new</span>&nbsp;Singleton();</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color: #ff3399;">return</span>&nbsp;uniqueInstance;</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">&nbsp;&nbsp;&nbsp;&nbsp;}</div><div style="padding: 0 6px; white-space: pre; line-height: 130%;">}</div></div><div style="text-align: right; margin-top: -13px; margin-right: 5px; font-size: 9px; font-style: italic;"><a style="color: #e5e5e5text-decoration:none;" href="http://colorscripter.com/info#e" target="_blank" rel="noopener">Colored by Color Scripter</a></div></td><td style="vertical-align: bottom; padding: 0 2px 4px 0;"><a style="text-decoration: none; color: white;" href="http://colorscripter.com/info#e" target="_blank" rel="noopener"><span style="font-size: 9px; word-break: normal; background-color: #e5e5e5; color: white; border-radius: 10px; padding: 1px;">cs</span></a></td></tr></tbody></table>

인스턴스가 있는지 확인하고 없으면 동기화된 블럭으로 들어갑니다. 이렇게 처리하면 처음에만 동기화가 됩니다.

블럭으로 들어간 후에도 다시 한 번 변수가 null 인지 확인한 다음 인스턴스를 생성했습니다.

volatile 키워드를 사용해서 멀티스레딩을 쓰더라도 uniqueInstance 변수가 Singleton 인스턴스로 초기화 되는 과정이 올바르게 진행되도록 할 수 있다. 상세 이유는 밑 포스팅을 참조

[자바에서 volatile 이란?](https://bepoz-study-diary.tistory.com/160)