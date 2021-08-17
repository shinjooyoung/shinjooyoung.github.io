---
title:  "리팩터링, 테스팅, 디버깅"
excerpt: "모던 자바 인 액션 정리 "
categories:
  - Study
tag:
  - Java
  - Book
  - ModernJavaInAction
toc: false
---

### 가독성과 유연성 개선하는 리팩토링

##### 코드 가독성 개선
> 어떤 코드를 다른 사람도 쉽게 이해 할수 잇음을 의미

익명 클래스를 람다 표현식으로 리팩터링하기
>모든 익명 클래스를 람다 표현식으로 변환할 수있는 것은 아니다.

-  익명 클래스에서 사용한 this와 super는 람다 표현식에서는 다른 의미를 갖는다. 
  - 익명 클래스에서 this는 자기자신을 가리키지만 람다에서 this는람다를 감싸는 클래스를 가리킨다.
- 익명 클래스는 감싸고 있는 클래스의 변수를가릴 수 있다(섀도 변수), 하지만 밑에 코드는 가릴 수 없다.
  
``` java
int a = 10;
Runable r1 = () ->{
	int a = 2; // 컴파일 에러
	Sysout.out.println(a)
}; 
```
  
- 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.
  - 익명 클래스는 인스턴스화 할때 명시적으로 형식이 정해지는 반면 람다의 형식은 콘텍스트에 따라 달라진다.
  
``` java
infterface TasK{
	public void excute();
}
public static void doSomthing(Runable r) {r.run();}

public static void doSomthing(Task a) {a.execyte();}

doSomeThing(new Task() {
  public void execute() {
    System.out.println("do");
  }
});

// 어떤 것이 실행되어야 하는지 알수 없다. 
doSomeThing(() -> System.out.println("do"));

// 명시적 형변환(Task)을 통해 모호하믈 제거할수 있다.
doSomeThing((Task)() -> System.out.println("do")); 
```

##### 람다 표현식을 메서드 참조로 리팩터링하기
>람다 표현식 대신 메서드 참조를 이용하면 가독성을 높일 수 있다.

``` java
//람다식
inventory.sort(
(Apple a1, Appl a2) -> a1.getWeight().compareTo(a2.getWeight()));

//메서드 참조
inventory.sort(comparing(Apple::getWeight));
```

##### 명령형 데이터 처리를 스트림으로 리팩터링 하기
> 스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다. 쇼트서킷과 게으름이라는 강력한 최적화 분아니라 멀티코어 아키텍처를 활용할 수 있는 지름길을 제공한다.

``` java
// 기존 코드
List<String> disNames = new ArrayList<>();
for(Dish dish : menu) {
	if(dish.getCalories() > 300) {
		dishNames.add(dish.getName()).
	}
}

// 스트림
menu.parallelstream()
	 .filter(d->d.getCalories() > 300)
	 .map(Dish::getName)
	 .collect(toList());
```

##### 코드 유연성 개선
> 다양한 람다를 전달해서 다양한 동작을표현할 수 있다. 따라서 변화하는 요구상에대응할 수 있는 코드를 구현할 수 있다.


조건부 연기 시행
- 내장 자바 Logger 클래스를 사용하는 예제이다
  - isLoggabe 메서드가 클라이언트로 노출되고 메시지를 로깅할때마다 상태를 매번 확인해야한다.
  
``` java
//로그의 상태를 매번 확인해야 한다.
if (logger.isLoggable(Log.FINER)) {
    logger.finer("Problem: " + generateDiagnostic());
}
```

- 자바 8 에는 이와 같은 문제를 해결할 수 있도록 Supplier를인수로 갖는 오버로드된 log 메서드를 제공한다.
- log 메서드는 logger의수준이 적절하게 설저되어 있을때만 인수로 넘겨진 람다를 내부적으로 실행한다.

``` java
//log 메서드의 내부 구현 코드
public void log(Level level, Supplier<String> msgSupplier){
    if(logger.isLoggable(level)){
        log(level, msgSupplier.get()); // 람다 실행
    }
}


Logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic());
```
---

### 람다로 객체지향 디자인패턴 리패터링하기

##### 전략 패턴
> 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이다.

- 알고리즘을 나타내는 인터페이스(Strategy 인터페이스)
- 다양한 알고리즘을 나타내는 한개 이상의 인터페이스 구현(ConcreateStrategyA, ConcreateStrategyA)
- 전략 객체를 사용하는 한 개 이상의 클라이언트

``` java
// String 문자열을 검증하는 인터페이스
public interface ValidationStrategy {
    boolean execute(String s);
}

//인터페이스를 구현하는 클래스 1
public class IsAllLowerCase implements ValidationStrategy {
    public boolean execute(String s) {
        return s.matches("[a-z]+");
    }
}
//인터페이스를 구현하는 클래스 2
public class IsNumeric implements ValidationStrategy {
    public boolean execute(String s) {
        return s.matches("\\d+");
    }
}

// 검증 전략 활용
public class Validator {
    private final ValidationStrategy strategy;
    public Validator(ValidationStrategy strategy) {
        this.strategy = strategy;
    }

    public boolean validate(String s) {
        return this.strategy.execute(s);
    }
}

Validator numericValidator = new Validator(new IsNumeric());
boolean b1 = numericValidator.validate("aaaa"); // false

Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
boolean b2 = lowerCaseValidator.validate("bbbb"); // true

```

람다 표현식 사용
- 다양한 전략을 구현하는 새로운 클래스를 구현할 필요 없이 람다 표현식을 직접 전달하면 코드가 간결해진다.

``` java
Validator numericValidator = 
    new Validator((String s) -> s.matches("[a-z]+"));
boolean b1 = numericValidator.validate("aaaa"); // false

Validator lowerCaseValidator = 
    new Validator((String s) -> s.matches("\\d+"));
boolean b2 = lowerCaseValidator.validate("bbbb"); // true
```

###### 탬플릿 메서드
> 알고리즘의 개요를 제시한 다음에 알고리즘의 일부를 고칠 수 있는 유연함을 제공할때 사용한다.
> 간단하게 '이 알고리즘을 사용하고 싶은데 그대로는 안되고 조금 고쳐야 하는 상황'에 적합하다.

온라인 뱅킹 애플리케이션의 동작을 정의하는 추상 클래스다.

``` java
abstract class OnlineBanking {
    public void processCustomer(int id) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
    }

    abstract void makeCustomerHappy(Customer c);
}
```
processCustomer 메서드는 온라인뱅킹 알고리즘이 해야 할 일을 보여준다.
각각의 지점(상속받는 클래스)은 OnlineBanking 클래스를 상속받아 makeCustomerHappy 메서드가 원하는 동작을 수행하도록 구현(오버라이딩)할 수 있다.


람다 표현식 사용

``` java
public class OnlineBankingLamda {
    public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy.accept(c);
    }
}

new OnlineBankingLamda().processCustomer(1337, (Customer c) -> 
    System.out:println("Hello "+ c.getName());
```
람다를 활용시 OnlineBanking을 상속받지 않고 직접 랍다 표현시글 전달해서 다양한 동작을 추가할 수 있다.
---

##### 옵저버 패턴
> 어떤 이벤트가 발생했을때 한 객체(주제라 불리는)가 다른 객체 리스트(옵저버라 불리는)에 자동으로 알림을 보내야 한느 상황에서 옵저버 디자인 패턴을 이용한다.

예를 들어 다양한 신문 매체가 뉴스 트윗을 구독하고 있으며 특정 키워드를 포함하는 트윗이등록되면 알림을 받을 수 있다.

``` java
// 새로운 트윗이 있을때 주제가 호출할 수 있도록
// notifi라고 하는 하나의 메서드를 제공
interface Observer {
    void notify(String tweet);
}

// 트윗에 포함된 다양한 키워드에 다른 동작을 수행할 수 있는 여러 옵저버를 정의할 수 있다. 
class NYTimes implements Observer {
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("money")) {
            System.out.println("Breaking news in NY! " + tweet);
        }
    }
}

class Guardian implements Observer {
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("queen")) {
            System.out.println("Yet another new in London... " + tweet);
        }
    }
}

class LeMonde implements Observer {
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("wine")) {
            System.out.println("Today cheese, wine and news! " + tweet);
        }
    }
}

// 주제 Subject 인터페이스의 정의다. 
interface Subject { 
    void registerObserver(Observer o);
    void notifyObservers(String tweet);
}

// registerobserver 메서드로 새로운 옵저버를 등록한 다음 
// notifyObservers 메서드로 ㅌ윗의 옵저버에 이를 알린다.
class Feed implements Subject {
    private final List<Observer> observers = new ArrayList<>();

    // 새로운 옵저를 등록한다. 
    public void registerObserver(Observer o) {
        this.observers.add(o);
    }

    // 트윗을 등록한 옵저들에게 알린다. 
    public void notifyObservers(String tweet) {
        observers.forEach(o -> o.notify(tweet));
    }
}

Feed f = new Feed();

// 구독할 옵저버를 등록한다. 
f.registerObserver(new NYTimes());
f.registerObserver(new Guardian());
f.registerObserver(new LeMonde());

// 옵저버들에게 메시지를 전송한다. 
f.notifyObservers("The queen said her favorite book is Modern Java in Action!");
```


람다 표현식 사용

``` java
Feed f = new Feed();

f.registerObserver((String tweet) {
    if(tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY! " + tweet);
    }
});

f.registerObserver((String tweet) {
    if(tweet != null && tweet.contains("queen")) {
        System.out.println("Yet another new in London... " + tweet);
    }
});
```
위의 에제는 실행할 동자기 비교적 간단하므로 람다 표현식으로 불필요한 코드를 제거하는 것이 바람직하다.
하지만 옵저버가 상태를 가지며, 여러 메서드를 정의하는 등 복잡하다면 람다 표현식보다 기존의 클래스 구현 방식을 고수하는 것이 바람직할 수도 있다.
---

##### 의무 체인 패턴
> 작업 처리 객체의 체인(동작 체인 등)을 만들 때는   의무 체인 패턴을 사용한다. 한객체가 어떤 작업을 처리한 다음에 다음에 다른객체로 결과를전달하고, 다른 객체도 해야 할 작업을처리한 다음에 또 다른 객체로 전달하는  식이다.

일반적으로 다음으로 처리할 객체 정보를 유지하는 필드를포함하는 작업 처리 추상 클래스로 의무 처리 패턴을 구상한다.
작업 처리 객체가 자신의 작업을 끝냈으면 다음 작업 처리 객체로 결과를 전달한다.

``` java
public abstract class ProcessingObject<T> {
    protected ProcessingObject<T> successor;

    public void setSuccessor(ProcessingObject<T> successor) {
        this.successor = successor;
    }
    public T handle(T input) {
        T r = handleWork(input);
        if (successor != null) {
            return successor.handle(r);
        }
        return r;
    }

    abstract protected T handleWork(T input);
}

public class HeaderTextProcessing extends ProcessingObject<String> {
    public String handleWork(String text) {
        return "From Raoul, Mario and Alan: " + text);
    }
}

public class SpellCheckerProcessing extends ProcessingObject<String> {
    public String handleWork(String text) {
        return text.replaceAll("labda", "lambda");
    }
}

ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObject<String> p2 = new SpellCheckerProcessing();

p1.setSuccessor(p2);

String result = p1.handle("Aren't ladas really sexy?!!");
System.out.println(result); 
// From Raoul, Mario and Alan: Aren't lambda really sexy?!!
```

람다 표현식
작업 처리 객체를 Function<String, string>, 더 정확히 표현하자면 UnaryOperator<String> 형식의 인스턴스로 표현할 수 있다.
and then 메서드로 이들함수를 조합해서 체임을 만들 수 있다.

``` java
// 첫번째 작업 처리 객체
UnaryProcessingObject<String> headerProcessing = 
    (String text) -> "From Raoul, Mario and Alan: " + text; 

// 두번째 작업 처리 객체
UnaryProcessingObject<String> spellCheckerProcessing = 
    (String text) -> text.replaceAll("labda", "lambda");
    
// 동적 체인으로 두함술로 조합
Function<String, String> pipeline = 
    headerProcessing.andThen(spellCheckerProcessing); 

String result = pipeline.apply("Aren't ladas really sexy?!!");
```
---




