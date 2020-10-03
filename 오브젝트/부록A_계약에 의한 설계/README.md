# 계약에 의한 설계
**계약에 의한 설계(Design By Contract, DBC)**를 사용하면 협력에 필요한 다양한 제약과 부수효과를 명시적으로 정의하고 문서화할 수 있다.  
## 협력과 계약  
* 협력에 참여하는 각 객체는 계약으로부터 **이익**을 기대하고 이익을 얻기 위해 **의무**를 이행한다.
* 협력에 참여하는 각 객체의 이익과 의무는 객체의 인터페이스 상에 **문서화**된다.  

계약에 의한 설계를 이용하면 오퍼레이션의 시그니처를 구성하는 다양한 요소들을 이용해 협력에 참여하는 객체들이 지켜야 하는 제약 조건을 명시할 수 있다.  

`public Reservation reserve( Customer customer, int audienceCount)`  
위를 통해 우리는 메서드의 이름과 매개변수의 이름을 통해 오퍼레이션이 클라이언트에게 어떤 것을 제공하는지를 충분히 설명할 수 있다.  

이 메서드는 예약 정보를 생성하는 것이기 때문에 한 명 이상의 예약자에 대해 예약 정보를 생성해야 한다.  
따라서 customer는 null이어서는 안 되고 audienceCount의 값은 1 이상이어야 한다.  

* **사전조건** : 메서드가 호출되기 위해 만족돼야 하는 조건. 이것은 메서드의 요구사항을 명시한다. 사전조건이 만족되지 않을 경우 메서드가 실행돼서는 안 된다.
사전조건을 만족시키는 것은 메서드를 실행하는 클라이언트의 의무다.  
* **사후조건** : 메서드가 실행된 후에 클라이언트에게 보장해야 하는 조건. 클라이언트가 사전조건을 만족시켰다면 메서드는 사후조건에 명시된 조건을 만족시켜야 한다.
만약 클라이언트가 사전조건을 만족시켰는데도 사후조건을 만족시키지 못한 경우에는 클라이언트에게 예외를 던져야 한다.
사후조건을 만족시키는 것은 서버의 의무다.  
* **불변식** : 항상 참이라고 보장되는 서버의 조건. 메서드가 실행되는 도중에는 불변식을 만족시키지 못할 수도 있지만 메서드를 실행하기 전이나 종료된 후에 불변식은 항상 참이어야 한다.  

위 3가지 요소에 **계약에 의한 설계**가 대응된다.  

```java
public Reservation Reserve(Customer customer, int audienceCount){
    contract.Requires(customer != null);
    contract.Requires(audienceCount >= 1);
    return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
}
```
Code Contracts에서 제공하는 대부분의 메서드는 System.Diagnostics.Contracts 네임스페이스의 static 클래스인 Contract가 제공한다.  
클라이언트가 사전조건을 만족시키지 못할 경우 Reserve 메서드는 최대한 빨리 실패해서 클라이언트에게 버그가 있다는 사실을 알린다.  
Contract.Requires 메서드는 클라이언트가 계약에 명시된 조건을 만족시키지 못할 경우 `ContractException` 예외를 발생시킨다.  

이 예제는 계약을 일반 로직과 분리해서 서술함으로써 계약을 좀 더 두드러지게 강조한 것을 볼 수가 있다.  
***
사후조건은 메서드의 실행 결과가 올바른지를 검사하고 실행 후에 객체가 유효한 상태로 남아있는지를 검사한다.  
일반적으로 사후조건은 다음과 같은 3 가지 용도로 사용된다.  
* 인스턴스 변수의 상태가 올바른지를 서술하기 위해
* 메서드에 전달된 파라미터의 값이 올바르게 변경됐는지를 서술하기 위해
* 반환값이 올바른지를 서술하기 위해  

Code Contracts에서 사후조건을 정의하기 위해서는 Contracts.Ensures 메서드를 제공한다.  
```java
public Reservation Reserve(Customer customer, int audienceCount){
    contract.Requires(customer != null);
    contract.Requires(audienceCount >= 1);
    contract.Ensures(Contract.Result<Reservation>()!=null);
    return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
}
```
Ensures는 하나 이상의 종료 지점이 나올 때에 유용하다.  
```java
public decimal Buy(Ticket ticket){
    if(bag.invited){
        bag.Ticket=ticket;
        return 0;
    }else{
        bag.Ticket=ticket;
        bag.MinusAbount(ticket.Fee);
        return ticket.Fee;
    }
}
```
초대장이 있을 경우에는 0원, 초대장이 없을 경우에는 티켓의 요금을 반환하는 해당 메서드rk aksdir Code Contract를 사용하지 않는다면  
사후조건을 두 개의 return 문 모두에 중복해야 작성했을 것을 아래와 같이 표기하면 1번이면 된다.   
```java
public decimal Buy(Ticket ticket){
    Contract.Requires(ticket!=null);
    Contract.Ensures(Contract.Result<decimal>()>=0);
    if(bag.invited){
        bag.Ticket=ticket;
        return 0;
    }else{
        bag.Ticket=ticket;
        bag.MinusAbount(ticket.Fee);
        return ticket.Fee;
    }
}
```
***
불변식은 다음과 같은 두 가지 특성을 가진다.  
* 불변식은 클래스의 모든 인스턴스가 생성된 후에 만족돼야 한다. 이것은 클래스에 정의된 모든 생성자는 불변식을 준수해야 한다는 것을 의미한다.  
* 불변식은 클라이언트에 의해 호출 가능한 모든 메서드에 의해 준수돼야 한다. 메서드가 실행되는 중에는 객체의 상태가 불안정한 상태로 빠질 수 있기 때문에 
불변식을 만족시킬 필요는 없지만 메서드 실행 전과 메서드 종료 후에는 항상 불변식을 만족하는 상태가 유지돼야 한다.  

불변식은 클래스의 모든 메서드의 사전조건과 사후조건에 추가되는 공통의 조건으로 생각할 수 있다.  
Code Contracts에서는 Contract.Invariant 메서드를 이용해 불변식을 정의할 수 있다.  
불변식은 생성자 실행 후, 메서드 실행 전, 메서드 실행 후에 호출돼야 하고,다행히 Code Contracts는 ContractInvariantMethod 애트리뷰트가  
지정된 메서드를 불변식을 체크해야 하는 모든 지점에 자동으로 추가한다.  
```java
public class Screening{
    private Movie movie;
    private int sequence;
    private DateTime whenScreened;
    
    [ContractInvariantMethod]
    private void Invariant(){
        Contract.Invariant(movie!=null);
        Contract.Invariant(sequence>=1);
        Contract.Invariant(whenScreened>DateTime.Now);
    }
}
```
***
## 계약에 의한 설계와 서브타이핑  
리스코프 치환 원칙의 규칙을 **계약 규칙**과 **가변성 규칙**으로 나눌 수 있는데,  
**계약 규칙**은  
* 서브타입에 더 강력한 사전조건을 정의할 수 없다.
* 서브타입에 더 완화된 사후조건을 정의할 수 없다.  
* 슈퍼타입의 불변식은 서브타입에서도 반드시 유지돼야 한다.  

**가변성 규칙**은
* 서브타입의 메서드 파라미터는 반공변성을 가져야 한다.
* 서브타입의 리턴타입은 공변성을 가져야 한다.
* 서브타입은 슈퍼타입이 발생시키는 예외와 다른타입의 예외를 발생시켜서는 안 된다.  

와 같은 특징을 가지고 있다.  