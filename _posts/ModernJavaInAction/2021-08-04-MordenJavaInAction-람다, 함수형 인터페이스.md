---
title:  "람다, 함수형 인터페이스"
excerpt: "모던 자바 인 액션 "
categories:
  - Study
tag:
  - Java
  - ModernJavaInAction
toc: true
---

##### 람다 표현식
- 한번 이상 실행할 수 있도록 전달할수 있는 코드 블록이다.
- 람다 표현식의 결과 타입은 지정하지 안흔다 결과 타입은 항상문맥으로 부터 추정된다.
- 람다 표현식이 어떤 경우에는 값을 리턴하고, 다른 경우에는 리턴하지 않는 것은 규칙에 어긋난다.
- 문법
  - 단순한 코드 블록으로 변수들의 명세와 함꼐 코드에 전달해야 한다.
```java
(String first, String second) -> Integer.compare(first.length(), second.length())
```
  - 표현식 하나로는 표현할 수 없는 계산을 수행한다면, 메서드를 작성할 때처럼 하면 된다.
```java
(String first, String second) -> {
  if(first.length() < second.length()) {
    return -1
  } else {
    return 0;
  }
}
```
  - 파라미터를 받지 않으면 파라미터 없는 메서드와 마찬가지로 빈 괄호를 사용한다.
```java
() -> Integer.compare(for(...))
```

  - 컴파일러는 파라미터 타입을 추정할 수 있는 경우에는 타입을 생략할 수 있다.
```java
(first, second) -> Integer.compare(first.length(), second.length())
```


  - 메서드에서 추정되는 타입 한 개를 파라미터로 받으면 괄호를 생략할 수 있다.
```java
EventHandeler<ActionEvent> listener = event -> Sysout.println("..");
```

  - 메서드 파라미터와 마찬가지 방식으로 파라미터에 애너테이션이나 final 수정자를 붙일 수 있다.
```java
(final String name) -> ...;
(@Notnull String name) -> ...;
```
- 람다 변수
  - 자유변수
    - 람다 표현식에서는익명 함수가 하는 것처럼 자유 변수(파라미터로 넘겨진 벼수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 
    - 이와 같은 동작은 람다 캡처링이라 부른다.
    - 정적 변수와 인스턴스 변수를 자유롭게 캡처 할수 있지만 지역 변수는 명시적으로 final로 선언되어 있어야 하거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.
    - 람다 표현식은 한번만 할당할 수 있는 지역 변수를 캡처할 수 있다.
    - 인스턴스 번수는 힙에 저장되는 반면 지역변수는 스택에 위치한다. 람다에서 바로 접근시 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다. 그래서 자바는 자유 지역 변수의 복사본을 제공하는데 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한번만 값을 할당해야 하는 제약이 생긴다.
    - 인스턴스 변수는 스레드가 공유하는 힙에 존재하므로 특별한 제약이 없다.


##### 함수형 인터페이스
- 메소드를 일급객체로 사용할 수 없는걸 보안하기 위해 사용된다
  - 일급 객체 : 다른 객체들에 일반적으로 적용 가능한 연산을 모두 지원하는 객체를 가르킨다. 보통 함수에 매개변수로 넘기기, 수정하기, 변수에 대입하기와 같은 연산을 지원할때 일급객체이다.
- 함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스다.
- 자바 API의 함수형 인터페이스로 Comparator, Runnable 등이 있는데  이러한 기존 인터페이스와 호환된다.
- 많은 디폴트 메서드를 가진 인터페이스라도 추상 메서드가 오직 하나면 함수형 인터페이스이다.
- 함수형 인터페이스는 추상 메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급(기숙적으로 따지면 함수형 인터페이스를 구현한 클래스의 인스턴스)라고 할수 있다.
- 함수형 인터페이로 변환이 자바에서 람다 표현식을 이용할 수 있는 유일한 일이다.
- 자바 API는 java.util.function 패키지에 다수의 아주 범용적인 함수형 인터페이스를 정의하고 있고 함수형 인터페이스에 @Functionalinterface 에너테이션을 붙일 수 있다.
- @FunctionallInterface 
  - 함수형 인터페이스임을 가리키는 어노테이션이다.
  - 인터페이스를 선언했지만 실제로 함수형 인터페이스가 아니면 컴파일 에러가 발생한다. 
- 함수형 인터페이스 검사 예외 문제
  - 함수형 인터페이스의 인스턴스로 변환될 때 검사 예외가 문제가 된다.
  - 람다 표현식의 몸체에서 검사 예외를 던질 수 있는 경우, 해당 예외가 대상 인터페이스의 추상 메서드에 선언되어 있어야 한다.
- 함수 디스크립터
  - 함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다.
  - 람다 표현식의 시그니처를 서술하는 메서드를 함수 디스크립터라고 부른다.
  
##### JAVA에서 사용되는 함수형 인터페이스
- 실행 어라운드 패턴
  - 자원처리의 사용하는 순환패턴은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다.
  - 실제 자원을 처리하느 ㄴ코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는걸 실행 어라운드 패턴이라고 부른다.
  - 직접 코드에 적용해봐야 완전히 알 것 같음 대충 느낌만 옴

- Predicate 
  - 인터페이스는 test라는 추상 메서드를 정의하며 제네릭 형식 T의 객체를 인수로 받아 블리언을 반환한다.
  - 순서 
    - Prdicate 인터페이스를 만든다.
    -제네릭 형태로 리스트와 Predicate를 받아 결과값을 추가하는 메서드를 만든다.
    - prideicate 객체를 람다 표현식으로생성 후 filter 메서드를 호출한다.

- Consumer
  - 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다. 
  - T형태의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 Consumer인터페이스를 사용할 수 있다.
  - 순서
    - Consumer 인터페이스를 만든다.
    - forEach 메서드를 만들고 제네릭 형태로 리스트와 Consumer 를 받아 동작을 수행하는 메서드를 만든다.
    - consumer 객체를 람다 표현식으로생성 후 forEach 메서드를 호출한다.

- Function
  - 인터페이스는 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 추상 메서드 apply를 정의한다.
  - 입력을 출력으로 매핑하는 람다를 정의할때 Function인터페이스를 활용할 수 있다. EX) 사과의 무게 정보를 추출하거나 문자열을 길이와 매핑
  - 순서
    - Function 인터페이스를 만든다<T,R>.
    - forEach 메서드를 만들고 제네릭 형태로 리스트와 Function 를 받아 동작을 수행하는 메서드를 만든다.
    - function 객체를 람다 표현식으로생성 후 map 메서드를 호출한다.

- 기본형 특화
  - 제네릭 파라미터는 참조형만 사용할 수 있다. 제네릭 내부 구현 때문에 어쩔 수 없는 일이다.
  - 자바에서는 기본형을 참조형으로 변환하는 기능을 제공하는데 이 기능을 박싱이라고 하고 반대는 언박싱이다.
  - 프로그래머가 편리하게 코드를 구현할 수 있도록 박싱과 언박싱이 자동으로 이루어지는 오ㅗ박싱이라는 기능도 제공한다.
  - 자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱을 피할  수 있도록 Int,Double,Long,In 이 앞에 붙고 뒤에 Predicate, Consumer, BinaryOperator, Function으로 사용하면 오토 박시없이 동작한다.
 


- 람다 사용 인터페이스 예제
  - 블리언 표현
    - Predicate<List<String>> EX) (List<String> list) -> list.isEmpty()
  - 객체 생성
    - Supplier<Apple> EX) () -> new Apple (10)
  - 객체에서 소비 
    - consumer<Apple> EX> (Apple a) -> System.out.println(a.getWeight())
  - 객체에서 선택/추출
    - Function<String,Integer>  OR ToIntFunction<String> EX) (String s) -> s.length()
  - 두 값 조합
    - IntBinaryOperator EX) (int a, int b) -> a * b
  - 두 객체 비교
    - Comparator<Apple> OR BiFunction<Apple, Apple, Integer> OR ToIntBiFunction<Apple,Apple> EX) (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())


##### 메서드 참조
- 메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.
  - 람다 : inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
  - 메서드 참조 : inventory.sort(comparing(Apple::getWeight));
- 메서드 참조는 특정 메서드만을 호출하는 람다의 축약형이라고 할 수있다.
- 메서드를 호출할때 메서드를 어떻게 호출해야 하는지 설명을 참조하기 보다는 메서드명을 직접 참조하는 것이 편리하다.
- 명시적으로 메서드명을 참조함으로 가독없다.높일 수 있고 실제로 호출하는 것이 아니므로 괄호는 필요가 없다.
- 정적 메서드 참조 : Integer::parseInt
- 다양한 형식의 인스턴스 메서드 참조 : String::length
  - (String s) -> s.toUpperCase() : String::toUpperCase
- 기존 객체의 인스턴스 메서드 참조 : 객체할당 지역변수::객체메서드

## 생성자 참조
- ClassName::new 처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를만들수 있다.
  - Supplier<Apple> c1 = Apple::new
  - Apple a1 = c1.get(); 
  - get 메서드를 호출해서 새로운 Apple객체를 만들 수 있다.
- 시그니처를 갖는 생성자는 Function 인터페이스의 시그니처와 같다
  - Function<Integer, Apple> c2 = Apple::new;
  -Apple a2 = a2.apply(110)
  - Function의 apply 메서드에 무게를 인수로 호출해서 새로운 Apple 객체를 만들 수 있다.

## 람다, 메서드 참조 활용하기
- inventory.sort(comparing(Apple::getWeight));
- 1단계 : 코드 전
  - Comparator를 implement한 클래스(AppleComparator)를 만들어 비교 메소드를 생성한다
  - inventory.sort(new AppleComparator());
- 2단계 : 익명 클래스 사용
  - 한번만 사용할 Comparator를 위 코드처럼 구현하는 것보다는 익명 클래스를이용하는 것이 좋다.
  - inventory.sort(new Comparator<Apple>(){public int compare(Apple a1, Apple a2){...}});
- 3단계 : 람다 표현식 사용
  - 자바 8에서는 람다 표현식이라는 경량화된 문법을 이용해서 코드를 전달할 수 있다.
  - 함수형 인터페이스를 기대하는 곳 어디에서나 람다 표현식을 사용할 수 있다.
  - 함수형 인터페이스란 오직 하나의추상 메서드를 정의하는 인터페이스로 추상 메서드의 시그니처(함수 디스크립터)는 람다 표현식의 시그니처를 정의한다. 
  - Comparator의 함수 디스크립터는 (T, T) -> int 다.
  - inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
  - 자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 활용해서 람다의 파라미터 형식을 추론하여 Apple을 제외할 수 있다.
  - Comparator는 Comparable키를 추출해서 Comparator 객체로 만든느 Function 함수를 인수로 받느 정적 메서드 comparing을 포함한다.
  - 다음처럼 사용가능(람다 표현식은 사과를 비교하는 데 사용할 키를 어떻게 추출할 것인지 지정하느 한개의 인수만 포함한다  ** 좀더 찾아보기 잘 이해 안감)
  - Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
- 4단계 : 메서드 참조 사용
  - 메서드 참조를 이용하면 코드를조금더 간소화하여 람다 표현식의 인수를 더 깔끔하게 전달할 수 있다.
  - inventory.sort(comparing(Apple::getWeight));
  - 최적의 코드로 코드만 짧아진게 아닌 의미도 명확해진다.
  - 코드 자체로 Apple을 weight별로 비교해서 inventory를 sort하라는 의미로 전달할수 있다.
- 람다 표현식을 조합할 수 있는 유용한 메서드
  - 자바 API의 몇몇 함수형 인터페이스는 다양한 유틸리티 메서드를 포함한다.
  - Comparator, Function, Predicate 같은 함수형 인터페이스는 람다 표현식을 조함할 수 있도록 유틸리티 메서드를 제공한다.
  - 메서드가 하나밖에 없는데 이를 가능하게 하는 이유는 디폴드 메서드 이다.