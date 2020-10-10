# Template Method Pattern
본 내용은 Head First Design Patterns 책을 토대로 정리하였습니다.

커피를 만드는 방법  
1. 물을 끓인다.
2. 끓는 물에 커피를 우려낸다.
3. 커피를 컵에 따른다.
4. 설탕과 우유를 추가한다.

차를 만드는 방법
1. 물을 끓인다.
2. 끓는 물에 차를 우려낸다.
3. 차를 컵에 따른다.
4. 레몬을 추가한다.

다음을 코드로 구현하게 된다면 1,3번이 중복될 코드가 될 것이다.  
간단하게 추상을 해서  

CaffeinBeverage  
prepareRecipe()   
boilWater()  
pourInCup()  

Coffee  
prepareRecipe()  
brewCoffeeGrinds()  
addSugarAndMilk()  

Tea  
prepareRecipe()  
steepTeaBag()  
addLemon()  

다음과 같이 나타냈다고 하자. 2,4번 항목은 추상화되지 않았지만 똑같다. 다른 음료에 적용될 뿐이다. prepareRecipe()를 어떻게 추상화 할까?  
Coffee와 Tea를 비교해보면 brewCoffeeGrinds()와 steepTeaBag()은 사실 거의 비슷하므로 brew()라는 메서드로 정리하고,  
addSugarAndMilk()와 addLemon() 또한 무언가를 추가한다는 것 자체는 같으니 addCondiments() 라는 메서드로 정리해보자  

```java
public abstract class CaffeineBeverage{
    final void prepareRecipe(){
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }

    abstract void brew();
    abstract void addCondiments();

    void boilWater(){
        System.out.println("물 끓이는 중");
    }

    void pourInCup(){
        System.out.println("컵에 따르는 중");
    }
}

public class Tea extends CaffeineBeverage{
    public void brew(){
        System.out.println("차를 우려내는 중");
    }
    public void addCondiments(){
        System.out.println("레몬을 추가하는 중");
    }
}

public class Coffee extends CaffeineBeverage{
    public void brew(){
        System.out.println("필터로 커피를 우려내는 중");
    }
    public void addCondiments(){
        System.out.println("설탕과 커피를 추가하는 중");
    }
}
``` 
prepareRecipe() 이라는 과정 자체는 오버라이딩 당하지 않게 final을 붙여 주었다.  
해당 brew()와 addCondiments()를 이제 Tea,Coffee 클래스에서 구현해주었다.  

지금까지 한 것이 ``템플릿 메서드 패턴``이다. prepareRecipe()가 바로 템플리 메서드이다.  
그 이유는 prepareRecipe()이 메서드이고, 어떤 알고리즘에 대한 템플릿 역할을 한다. 이 경우에는 카페인이 들어있는 음료를 만들기 위한 템플릿이다.  
템플릿 내에서 알고리즘의 각 단계는 메서드로 표현이 되고, 어떤 메서드는 이 클래스 내에서, 어떤 것은 서브 클래스에서 처리된다.  

처음에 만들었던 Tea및 Coffee와 클래스와 템플릿 메서드 패턴을 적용한 Tea와 Coffee 클래스를 비교해보면,  
Coffee와 Tea가 각각 작업을 처리 => CaffeineBeverage 클래스에서 작업을 처리  
Coffee와 Tea에 중복 코드 존재 => CaffeineBeverage덕에 서브클래스에서 코드 재사용 가능  
알고리즘이 바뀌면 서브클래스 수정 필요 => 알고리즘이 한 군데에 모여 있기 때문에 그 부분만 고치면 됨  
새로운 음료 추가시 많은 작업 필요 => 몇 가지 메서드만 추가하면 됨  
알고리즘에 대한 지식과 구현 방법이 여러 클래스에 분산 => CaffeineBeverage 클래스에 알고리즘 집중되어 있어 일부 구현만 서브클래스에 의존  

>**템플릿 메서드 패턴**에서는 메서드에서 알고리즘의 골격을 정의합니다. 알고리즘의 여러 단계 중 일부는 서브클래스에서 구현할 수 있습니다.
>템플릿 메서드를 이용하면 알고리즘의 구조는 그대로 유지하면서 서브클래스에서 특정 단계를 재정의할 수 있습니다.

후크(hook)는 추상 클래스에서 선언되는 아무 코드도 들어있지 않는 메서드이다. 이것을 이용해서 서브클래스 입장에서 다양한 위치에서 알고리즘에 끼어들 수가 있게 된다.  
```java
public class CoffeeWithHook extends CaffeineBeverageWithHook{
    public void brew(){
        System.out.println("필터로 커피를 우려내는 중");
    }
    public void addCondiments(){
        System.out.println("설탕과 커피를 추가하는 중");
    }
    
    public boolean customerWantsCondiments(){
        String answer=getUserInput();
        if (answer.toLowerCase().startsWith("y")) {
            return true;
        } else {
            return false;
        }
    }
    
    private String getUserInput(){
        String answer=null;
        System.out.println("커피와 우유와 설탕을 넣어 드릴까요? (y/n)");

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        try {
            answer = br.readLine();
        } catch (IOException e) {
            System.err.println("IO 오류");
        }
        if (answer == null) {
            return "no";
        }
        return answer;
    }
}

public class BeverageTestDrive{
    public static void main(String[] args) {
        TeaWithHook teaHook = new TeaWithHook();
        CoffeeWithHook coffeeHook = new CoffeeWithHook();

        System.out.println("\n차 준비중...");
        teaHook.prepareRecipe();
        System.out.println("\n커피 준비중...");
        coffeeHook.prepareRecipe();
    }
}
```
다음과 같이 말이다.  
템플릿 메서드 패턴은 프레임워크에서 자주 볼 수 있다. 자바 API에서도 찾아볼 수가 있다.  
정렬을 하기 위해 사용하는 Arrays 클래스의 sort가 바로 그렇다.  
```java
public static void sort(Object[] a){
    Object[] aux = (Object[])a.clone();
    mergeSort(aux, a, 0, a.length, 0);
}

private static void mergeSort(Object[] src, Object[] dest, int low, int high, int off){
    for(int i=low;i<high;i++){
        for(int j=i;j>low && ((Comparable)dest[j-1]).comparTo((Comparable)dest[j])>0;j--){
            swap(dest,j,j-1);
        }
    }
    return;
}
```
mergeSort가 바로 템플릿 메서드이다. compareTo() 메서드를 구현해주어야만 하고, swap메서드는 이미 정의되어 있는 구상 메서드이다.  
compareTo 메서드는 Comparable 인터페이스를 클래스에서 받아서 구현해줄 수가 있다.  
이 예제는 교과서적인 템플릿 메서드라고 할 수는 없겠지만, 메서드 자체는 템플릿 메서드 패턴의 기본 정신을 충실하게 따르고 있다.  
이외에도 InputStream에 있는 read() 메서드는 서브클래스에서 구현해줘야 하며 read(byte[] b,int off,int len) 템플릿 메서드에서 쓰인다.  
***