본 내용은 Head First Design Patterns 책을 토대로 정리하였습니다.

"new"는 "구상 객체"를 뜻한다.

특정 구현을 바탕으로 프로그래밍을 하는것은 지양해야 하는데 new 를 쓸 때마다 특정 구현을 사용하게 된다.

만약 인터페이스에 맞춰서 코딩을 하게되면 다형성 덕분에 어떤 클래스든 특정 인터페이스만 구현하면 사용할 수 있게 된다. 반대로 코드에서 구상 클래스를 많이 사용하면 새로운 구상 클래스가 추가될 때마다 코드를 고쳐야 하기 때문에 문제가 발생하게 된다. 즉 변화에 대해 닫혀있는 코드가 만들어 지게된다. 

**우리는 확장에 대해서는 열려 있고 변경에 대해서는 닫혀있어야 하는 OCP를 지향해야 한다.**

```java
    Pizza orderPizza(){
        Pizza pizza = new Pizza();
        
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
 
    Pizza orderPizza(String type){
        Pizza pizza;
 
        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("greek")) {
            pizza = new GreekPizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        }
        
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
```

위의 Pizza 클래스는 아래의 Pizza 클래스로 개선할 수 있을 것이다.

피자 종류를 바탕으로 올바른 구상 클래스의 인스턴스를 만들고 그 인스턴스를 pizza 인스턴스 변수에 대입한다.

하지만, 피자메뉴가 추가되거나 삭제 될 때에 인스턴스를 대입하는 과정의 코드를 계속해서 고쳐야 한다.

이 부분에서 코드 변경에 대해 닫혀있지 않다는 것을 알 수가 있다.

해당 객체 생성 부분은 따로 캡슐화를 시킬 것이다.

해당 코드는 피자를 만드는 일만 처리하는 객체에 넣을 것이다. 그리고 해당 객체의 이름은 팩토리라고 붙일 것이다.

기존의 orderPizza메서드에는 pizza인터페이스를 구현하는 피자를 받아서 기존에 정의되어 있는 prepare...box메서드만

호출하면 된다.

```java
    public class SimplePizzaFactory{
        public Pizza createPizza(String type) {
            Pizza pizza = null;
            if (type.equals("cheese")) {
                pizza = new CheesePizza();
            } else if (type.equals("clam")) {
                pizza = new ClamPizza();
            } else if (type.equals("pepperoni")) {
                pizza = new PepperoniPizza();
            } else if (type.equals("veggie")) {
                pizza = new VeggiePizza();
            }
            return pizza;
        }
    }
```

SimplePizzaFactory 클래스로 객체 생성 부분을 전담하였다.   
이를 통해, 여러 클래스에서 피자를 생성하는 작업을 따로 캡슐화 시켜주었다.

이제 클라이언트 코드를 해당 팩토리 코드를 이용해 작성해 보겠다.

```java
    public class PizzaStore{
        SimplePizzaFactory factory;
        
        public PizzaStore(SimplePizzaFactory factory){
            this.factory = factory;
        }
        public Pizza orderPizza(String type){
            Pizza pizza = factory.createPizza(type);
            
            pizza.prepare();
            pizza.bake();
            pizza.cut();
            pizza.box();
            return pizza;
        }
    }
```

pizza를 생성하는 과정에서 new 대신 팩토리 객체에 있는 create 메서드를 사용하였다. 

현재까지 진행한 Simple Factory는 사실 디자인 패턴이라고 불리기는 어렵다. 프로그래밍을 하는데 있어서 자주 쓰이는 관용구에 가깝다.

만약 PizzaStore가 분점을 내서 뉴욕에도 생기고 시카고에도 생겼다고 할 때에 어떻게 처리해야 할까?

SimplePizzaFactory 에서 했던 것 처럼 NYPizzaFactory, ChicagoPizzaFactory를 생성해서 받아오면 문제가 없을 것이다.

```java
        NYPizzaFactory nyFactory = new NYPizzaFactory();
        PizzaStore nyStore = new PizzaStore(nyFactory);
        nyStore.order("Veggie");
        
        ChicagoFactory chicagoFactory = new ChicagoFactory();
        PizzaStore chicagoStore = new PizzaStore(chicagoFactory);
        chicagoStore.order("Veggie");
```

다음과 같이 말이다. 

하지만, 분점마다 피자 제작을 독자적으로 제작한다면 어떻게 해야될까?

```java
     public abstract class PizzaStore{
         public Pizza orderPizza(String type) {
             Pizza pizza;
             
             pizza= createPizza(type);
             
             pizza.prepare();
             pizza.bake();
             pizza.cut();
             pizza.box();
             
             return pizza;
         }
         
         abstract Pizza createPizza(String type);
     }
```

이제 createPizza를 팩토리 객체에 있는 createPizza가 아닌 PizzaStore내부에 있는 메서드를 사용하게 되었고

추상 메서드로 변환되었다.

![fac1](https://user-images.githubusercontent.com/45073750/93668547-deb41080-fac7-11ea-8503-abe8363eb9ed.png)

다음과 같은 형식으로 이루어지게 될 것이다.

```java
     public Pizza createPizza(type){
         if (type.equals("cheese")) {
             pizza=new NYStyleCheesePizza();
         }
     }
```

내부는 대충 이런식으로 각 지점별 피자가 만들어질 것이다. 

PizzaStore를 한 번 제대로 구현해보겠다.

```java
    public class NYPizzaStore extends PizzaStore{
         Pizza createPizza(String item){
             if(item.equals("cheese")){
                 return new NYStyleCheesePizza();
             } else if (item.equals("veggie")) {
                 return new NYStyleVeggiePizza();
             } else if (item.equals("clam")) {
                 return new NYStyleClamPizza();
             } else if (item.equals("pepperoni")) {
                 return new NYStylePepperoniPizza();
             } else return null;
             
         }
    }
```

Pizza 인스턴스를 만드는 일은 이제 팩토리 역할을 하는 메서드에서 맡아서 처리한다. 

**(protected abstract Pizza createPizza(String type))**

**팩토리 메서드는 객체 생성을 처리하며, 팩토리 메서드를 이용하면 객체를 생성하는 작업을 서브클래스에 캡슐화시킬 수 있다. 이렇게 하면 슈퍼클래스에 있는 클라이언트 코드와 서브클래스에 있는 객체 생성 코드를 분리시킬 수 있다.**

팩토리 메서드는 클라이언트(orderPizza()와 같은 코드)에서 실제로 생성되는 구상 객체가 무엇인지 알 수 없게 만드는 역할을 하기도 한다.

주문 과정을 따라가보면

1.  뉴욕 피자 가게 지점을 선택. PizzaStore nyPizzaStore = new NYPizzaStore();
2.  피자 가게가 확보 되었으니 주문을 함. nyPizzaStore.orderPizza("cheese");
3.  orderPizza()메서드에서 createPizza() 메서드를 호출함. Pizza pizza=createPizza("cheese");
4.  나머지 order 작업을 한다. pizza.prepare() ..... pizza.box();

이제 Pizza 클래스를 구현해보겠다.

```java
    public abstract class Pizza{
        String name;
        String dough;
        String sauce;
        ArrayList toppings = new ArrayList();
        
        void prepare(){
            System.out.println("Preparing "+name);
            System.out.println("Tossing dough...");
            System.out.println("Adding sauce...");
            System.out.println("Adding toppings: ");
            for (int i = 0; i < toppings.size(); i++) {
                System.out.println("    " + toppings.get(i));
            }
        }
        
        void bake() {
            System.out.println("Bake for 25 minutes at 350");
        }
        void cut(){
            System.out.println("Cutting the pizza into diagonal slices");
        }
        void box(){
            System.out.println("Place pizza in official pizzaStore box");
        }
        public String getName() {
            return name;
        }
    }
```

피자를 만들어보면

```java
    public class NYStyleCheesePizza extends Pizza{
         public NYStyleCheesePizza(){
             name = "NY Style Sauce and Cheese Pizza";
             dough = "Thin Crust Dough";
             sauce = "Marinara Sacue";
             
             toppings.add("Grated Reggiano Cheese");
         }
    }
```

다음과 같이 만들었다.

```java
    public class PizzaTestDrive{
        public static void main(String[] args) {
            PizzaStore nyStore = new NYPizzaStore();
            PizzaStore chicagoStore = new ChicagoPizzaStore();
 
            Pizza pizza = nyStore.orderPizza("cheese");
            System.out.println("Bepoz ordered a "+pizza.getName());
 
            pizza = chicagoStore.orderPizza("cheese");
            System.out.println("Mom orderd a " + pizza.getName());
        }
    }
```

최종적으로 실행시켜보았다.

**모든 팩토리 패턴에서는 객체 생성을 캡슐화 한다.**

**팩토리 메서드 패턴에서는 서브클래스에서 어떤 클래스를 만들지를 결정하게 함으로써 객체 생성을 캡슐화 한다.**

위의 예시에서 PizzaStore 추상 클래스와 NYPizzaStore, ChicagoPizzaStore는 생산자(creator) 클래스이다.

Pizza클래스와 이를 상속한 각종 피자들은 제품(Product) 클래스이다.

** 팩토리 메서드 패턴**

**\- 팩토리 메서드 패턴에서는 객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 인스턴스를 만들지는 서브클래스에서 결정하게 만든다. 팩토리 메서드 패턴을 이용하면 클래스의 인스턴스를 만드는 일을 서브클래스에게 맡기게 된다. **

![fac2](https://user-images.githubusercontent.com/45073750/93668581-25a20600-fac8-11ea-80e9-2be7224da57f.png)

다음과 같은 다이어그램으로 나타낼 수 있는데,

Creator에는 제품을 가지고 원하는 일을 하기 위한 모든 메서드들이 구현되어 있다. 하지만, 제품을 만들어 주는 팩토리 메서드는 추상메서드로 정의되어 있을뿐 구현되어 있지는 않다.

ConcreteCreator 에서 제품을 만드는 factoryMethod()의 구현이 이루어 진다.

제품 클래스에서는 모두 똑같은 인터페이스를 구현해야 한다. 그래야 그 제품을 사용할 클래스에서 구상 클래스가 아닌 인터페이스에 대한 래퍼런스를 써서 객체를 참조할 수 있기 때문이다.

> Q: ConcreteCreator가 하나 밖에 없는 경우에는 팩토리 메서드 패턴을 쓴다고 해도 별로 장점이 없지 않나요?  
>   
> A: 구상 생산자 클래스가 하나 밖에 없더라도 제품을 생산하는 부분과 사용하는 부분을 분리시킬 수 있으니 유용하다. 다른 제품을 추가하거나 제품 구성을 변경시키더라도 Creator 클래스가 ConcreteProduct와 느슨하게 결합되어 있으므로 Creator를 건드릴 필요가 없다.  
>   
>   
> Q: 팩토리 메서드와 생산자 클래스는 항상 추상으로 선언해야 하나?  
>   
> A: 꼭 그런것은 아니다. 구체적인 제품을 어느정도까지 기본적으로 만들어주는 기본 팩토리 메서드를 정의해도       된다. 그렇게 하면 Creator의 서브클래스를 만들지 않아도 제품을 만들어내도록 할 수 있다.  
>   
>   
> Q: 매개변수 팩토리 메서드를 사용하면 type-safety에 지장이 있지 않나요? clam을 calm으로 친다던가...  
>   
> A: 만약 그런일이 벌어진다면 런타임 오류가 발생하게 될 것이다. 매개변수 형식을 나타내기 위한 객체를 만들거      나, enum을 사용하거나 할 수도 있을 것이다.

장점을 정리해보자면,

객체 생성 코드를 전부 한 객체 또는 메서드에 집어넣어서 코드의 중복된 내용을 제거할 수가 있고, 나중에 관리할        때도 한 군데만 신경을 쓰면 된다. 클라이언트 입장에서는 객체 인스턴스를 만들 때 필요한 구상 클래스가 아닌 인터페이스만 필요로 하게된다.

객체 생성을 피할 수 없는 것은 맞지만, 생성 코드를 한 곳에 모아놓고 체계적으로 관리할 수 있는 디자인을 만드는 것을 지향한다. 이렇게 모아두면 객체 인스턴스를 만드는 코드를 보호하고 관리하기가 편해지기 때문이다.

이제 피자를 만드는데 들어가는 재료들을 대상으로 패턴으로 적용해보겠다. 해당 원재료 또한 원재료 공장이 지역마다 다를 것이다.

```java
    public interface PizzaIngredientFactory{
         public Dough createDough();
         public Sauce createSauce();
         public Cheese createCheese();
         public Veggies[] createVeggeis();
         public Pepperoni createPepperoni();
         public Clams createClams();
    }
    
    public class NYPizzaIngredientFactory implements PizzaIngredientFactory{
         public Dough createDough(){
             return new ThinCrushDough();
         }
 
        public Sauce createSauce() {
            return new MarinaraSauce();
        }
        
        public Cheese createCheese(){
             return new ReggianoCheese();
        }
        
        public Veggies[] createVeggies(){
             Veggies[] veggies={new Garlic(), new Onion(), new Mushroom(), new RedPepper()};
             return veggies;
        }
        
        public Pepperoni createPepperoni(){
             return new SlicedPepperoni();
        }
        
        public Clams createClam(){
             return new FreshClams();
        }
    }
```
원재료 공장을 만들고 뉴욕 원재료 공장도 구현해보았다.

```java
        void prepare(){
            System.out.println("Preparing "+name);
            System.out.println("Tossing dough...");
            System.out.println("Adding sauce...");
            System.out.println("Adding toppings: ");
            for (int i = 0; i < toppings.size(); i++) {
                System.out.println("    " + toppings.get(i));
            }
        }
 
        abstract void prepare();
```
이제 Pizza 클래스에서 팩토리에서 생산한 원재료만 사용하도록 코드를 고쳤다.

prepare 메서드를 추상 메서드로 변환 시켰다.

```java
    public class CheesePizza extends Pizza{
         PizzaIngredientFactory ingredientFactory;
 
        public CheesePizza(PizzaIngredientFactory ingredientFactory) {
            this.ingredientFactory=ingredientFactory;
        }
 
        void prepare(){
            System.out.println("Preparing "+name);
            dough = ingredientFactory.createDough();
            sauce = ingredientFactory.createSauce();
            cheese = ingredientFactory.createCheese();
        }
    }
```
Pizza 클래스를 상속받는 과정에서 ingredientfacotry를 넘겨주고 해당 재료로 prepare() 메서드를 구현하게된다.

```java
    public class NYPizzaStore extends PizzaStore{
        protected Pizza createPizza(String item) {
            Pizza pizza = null;
            PizzaIngredientFactory ingredientFactory = new NYPizzaIngredientFactory();
 
            if (item.equals("cheese")) {
                pizza = new CheesePizza(ingredientFactory);
                pizza.setName("New York Style Cheese Pizza");
            } else if (item.equals("veggie")) {
                pizza = new VeggiePizza(ingredientFactory);
                pizza.setName("New York Style Veggie Pizza");
            } else if (item.equals("clam")) {
                pizza = new ClamPizza(ingredientFactory);
                pizza.setName("New York Style Clam Pizza");
            } else if (item.equals("pepperoni")) {
                pizza = new PepperoniPizza(ingredientFactory);
                pizza.setName("New York Style Pepperoni Pizza");
            }
            return pizza;
        }
    }
```
NYPizzaStore를 구현해보았다.

createPizza 내부에서 NYPizzaIngredientFactory를 명시해주고 해당 원재료팩토리를 피자클래스를 구현할 때에 파라미터로 보내서 prepare를 재료에 원재료 공장에 알맞게 구현을 해준다는 것을 알 수가 있다.

그렇다면 다시 주문과정을 따라가보자. 아까전의 주문과정 3번과 4번 사이에 2개의 과정이 추가된다.

1.  뉴욕 피자 가게 지점을 선택. PizzaStore nyPizzaStore = new NYPizzaStore();
2.  피자 가게가 확보 되었으니 주문을 함. nyPizzaStore.orderPizza("cheese");
3.  orderPizza()메서드에서 createPizza() 메서드를 호출함. Pizza pizza=createPizza("cheese");
4.  Pizza pizza=new CheesePizza(nyingrefac); 원재료 공장을 파라미터로 넘긴다.
5.  void prepare(){ dough=fac.createDough()...cheese=fac.createCheese();}  공장에 따른 재료가 준비된다.
6.  나머지 order 작업을 한다. pizza.prepare() ..... pizza.box();

이 원재료 팩토리를 이용하는 방법을 추상 팩토리 패턴이라고 한다.

** 추상 팩토리 패턴**

**\- 추상 팩토리 패턴에서는 인터페이스를 이용하여 서로 연관된, 또는 의존하는 객체를 구상 클래스를 지정하지 않고도 생성할 수 있다.**

클래스 다이어그램으로 표현하면

![fac3](https://user-images.githubusercontent.com/45073750/93668607-58e49500-fac8-11ea-9650-aaab16ebea49.png)

다음과 같다. 꽤나 복잡하다.

팩토레 메서드 패턴과 추상 팩토리 패턴은 참 비슷해서 헷갈린다.

둘 다 객체를 만드는 일을 한다.

팩토리 메서드 패턴은 상속을 통해서 만들고, 추상 팩토리 패턴은 객체 구성을 통해서 만든다. 

  
팩토리 메소드 패턴은 서브클래스를 통해서 객체를 만든다. 그렇게 하면 클라이언트에서 자신이 사용할 추상 형식만 알면 되고, 구상 형식은 서브클래스에서 처리해준다. 바꿔서 말하면 클라이언트와 구상 형식을 분리시켜주는 역할이다.

추상 팩토리 패턴도 마찬가지이지만 방법이 다르다. 추상 팩토리 패턴은 제품군을 만들기 위한 추상 형식을 제공한다. 팩토리를 이용하고 싶으면 일단 인스턴스를 만든 다음 추상 형식을 써서 만든 코드에 전달하면 된다.

클라이언트에서 서로 연관된 일련의 제품들을 만들어야 할 때, 추상 팩토리 패턴을 활용한다.

클라이언트 코드와 인스턴스를 만들어야 할 구상 클래스를 분리시켜야 할 때에 팩토리 메서드 패턴을 활용한다.