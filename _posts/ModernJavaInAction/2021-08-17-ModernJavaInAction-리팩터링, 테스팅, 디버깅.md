---
title:  "리팩터링, 테스팅, 디버깅"
excerpt: "모던 자바 인 액션 "
categories:
  - Study
tag:
  - Java
  - ModernJavaInAction
toc: true
---


##### 코드 가독성 개선
> 어떤 코드를 다른 사람도 쉽게 이해 할수 잇음을 의미

### 익명 클래스를 람다 표현식으로 리팩터링하기

>모든 익명 클래스를 람다 표현식으로 변환할 수있는 것은 아니다.

- 익명 클래스에서 사용한 this와 super는 람다 표현식에서는 다른 의미를 갖는다. 
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


**조건부 연기 시행**  
내장 자바 Logger 클래스를 사용하는 예제이다. isLoggabe 메서드가 클라이언트로 노출되고 메시지를 로깅할때마다 상태를 매번 확인해야한다.
  
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

**람다 표현식**  
다양한 전략을 구현하는 새로운 클래스를 구현할 필요 없이 람다 표현식을 직접 전달하면 코드가 간결해진다.

``` java
Validator numericValidator = 
    new Validator((String s) -> s.matches("[a-z]+"));
boolean b1 = numericValidator.validate("aaaa"); // false

Validator lowerCaseValidator = 
    new Validator((String s) -> s.matches("\\d+"));
boolean b2 = lowerCaseValidator.validate("bbbb"); // true
```

##### 탬플릿 메서드
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


**람다 표현식**  

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
interface Subjecㅇt { 
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


**람다 표현식**

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

**람다 표현식**  
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

##### 팩토리 패턴

>인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.

**예시**  
다양한 상품을 만드는 Factory 클래스이다. 여기서 Loan, stock, Bond는 모두 product의 서브형식이다.

``` java
public class productFactory{
	public static product createProduct(String name){
		switch(name){
			case "long" : return new Loan();
			case "stock" : return new Stock();
			case "bond" : return new Bond();
			default : throw new RuntimeException("No such product" + name);
		}
	}
}

// 클라이언트
product p = productFactory.createProduct("loan");
```

**람다 표현식 사용**  
생성자도 메서드 참조처럼 접근할 수 있다. 

``` java
//Loan 생성자를 사용하는 코드이다.
Supplier<Product>Loan::new;
Loan loan = loanSupplier.get();
```

	상품명을 생성자로 연결하는 Map을 만들어 코드를 재구현 할 수 있다.  

``` java
//
final static Map<String, Supplier<Produt>> map = new HashMap<>();

static{
	map.put("loan", Loan::new);
	map.put("stock", Stock::new);
	map.put("bond", Vond::new);
}

public static Product createproduct(String name){
	Supplier<Product> p = map.get(name);
	if(p != null) return p.get();
	thirow new IllegalArgumentException("No such product" + name);
}

```

자바 8의 새로운 기능으로 깔끔하게 구현햇으나 여러 인수를전달하는 상황에서는 위 기법을 적용하기 어렵다.

##### 람다 테스팅

>일반적으로 좋은소프트웨어 공학자라면 프로그램이 의도대로 동작하는지 확인할 수 있는 단위 테스팅을진행한다.

**보이는 람다 표션식의 동작 테스팅**  
람다는 익명이므로 테스트 코드이름을 호출할수 없다. 필요하다면 람다를 필드에 저장해서 재사용 할 수 있으며 람다의 로직을 테스트할수 있다.  

``` java

public class Point{
	// 람다 표현식을 필드에저장
	// 테스트만을 위해서 필드에 선언한다면 오히려 불편해 보인다..
	public final static Comparator<point> compareByXAndThenY =
		comparing(Point::getx).thenComparing(Point::getY)
}

@Test 
public void testComparingTwoPoints() throws Exception { 
	Point p1 = new Point(10, 15); 
	Point p2 = new Point(10, 20); 
	int result = Point.compareByXAndThenY.compare(p1, p2); 
	assertTrue(result < 0);
}

```


**람다를 사용하는 메서드의 동작에 집중**  
람다의 목표는 정해진 동작을 다른 메서드에 사용할 수 있도록 하나의 조각으로 캡슐화하는 것이다. 그러려면 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다. 람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.

>위에처럼 필드에 람다식을 선언하는게 아닌 단순히 메서드를 테스트 함으로써 공개하지 않고 검증한다는 이야기 같다.

``` java
public static List<Point> moveAllPointsRightBy(List<Point> points, int x) {
    return points.stream()
                .map(p -> new Point(p.getX() + x, p.getY())
                .collect(toList());
}
```
위 코드에 람다 표현식 p -> new Point(p.getX() + x, p.getY()); 를 테스트하는 부분은 없다. 그냥 moveAllPointsRightBy 메서드를 구현한 코드일 뿐이다. 이제 moveAllPointsRightBy 메서드의 동작을 확인할 수 있다.

``` java
@Test
public void testMoveAllPointsRightBy() throws Exception {
    List<Point> points = Arrays.asList(new Point(5, 5), new Point(10, 5));
    List<Point> expectedPoints = Arrays.asList(new Point(15, 5), new Point(20, 5));
    List<Point> newPoints = Point.moveAllPointsRightBy(points,10);
    assertEquals(expectedPoints, newPoints);
}
```
위 단위 테스트에서 보여주는 것처럼 Point 클래스의 equals 메서드는 중요한 메서드다. 따라서 Object의 기본적인 equals 구현을 그대로 사용하지 않으려면 equals 메서드를 적절하게 구현해야 한다.


**복잡한 람다를 개별 메서드로 분할하기**  
람다 표현식은 참조할 수 없다, 테스트를 위해서는 표현식을 메서드로 만들어 메서드 참조로 바꾸는 것이다. 그러면 일반 메서드를 테스트하듯이 람다 표현식을 테스트할 수 있다.


**고차원 함수 테스팅**  
함수를 인수로 받거나 받거나 다른함수를 반환하는 메서드(고차원 함수)는 메서드가 람다를 인수로 받는다면 다른 람다로 메서드의동작을 테스트 할 수 있다.

``` java
@Test
public void testFilter() throws Exception {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    List<Integer> even = filter(numbers, i -> i % 2 == 0);
    List<Integer> smallerThanThree = filter(numbers, i -> i < 3);
    assertEquals(Arrays.asList(2, 4), even);
    assertEquals(Arrays.asList(1, 2), smallerThanThree);
}
```

##### 디버깅

>문제가 발생한 코드를 디버깅할때 개발자는 **스택 트레이스**, **로깅**을 가장 먼저 확인해야 한다.

**스택 트레이스**  
외 발생으로 프로그램 실행이 갑자기 중단되었다면 어디에서 멈췄고 어떻게 멈추게 되었는지를 스택 프레임에서 확인 가능하다.



**람다와 스택 트레이스 **  
람다 표현식은 이름이 없기 때문에 조금 복잡한 스택트레이스가 생성된다. 람다 표현식에서 에러가 발생시에는 lambda$main$0 표시된다.  
메서드 참조를 사용하는 경우에는 클래스와 같은 곳에 선언되어 있을때만 표시가 된다.


**정보 로깅**  
스트림의 파이프 라인에서 연산을 한는 중 forEach를 호출하면 호출한 순간 스트림은 소비가 된다. 각 연산의 결과들을 확인하려면 peek을 사용하면 된다. peek은 스트림을 소비한 것처럼 동작한다.

``` java
List<Integer> result = numbers.stream()
									.peak(x->System.out.println("from stream : " + x))
									.map(x->x+17)
									.peak(x->System.out.println("after map: " + x))
									.filter(x->x%2==0)
									.peak(x->System.out.println("after filter: " + x))
```



