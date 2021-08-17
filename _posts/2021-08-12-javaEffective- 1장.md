---
title:  "생성자 대신 정적 팩터리 메서드 고려"
excerpt: "JavaEffective 정리 "
categories:
  - Study
tag:
  - Java
  - Book
  - JavaEffective
toc: true
---


##### 장점

- 이름을 가질 수 있다 
  - 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
  - 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 경우 생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이가 잘 드러내는 이름을 지어주면 된다. 
  - 변수에 값을 선택적으로 넣어 인스턴스를 반환해야 할때 활용이 가능하다.
  
- 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
  - 인스턴스를 미리 만들어 놓거나 새로 생성한인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다. 특히 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려준다. 
  - static final을 통해 인스턴스를 생성하고 메서드 호출시 해당 인스턴스를 반환하는 방식으로 활용할 수 있다.

- 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. 
  - 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 엄청난 유연성을 선물한다. 
  - api를 만들때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할수 있어 api를 작게 유지할 수 있다. 
  - 객체 반환시 자식 타입으로 반환하여 자유롭게 선택이가능하다. 
   
  ``` java
	public class Parent {
		public Parent() {
		}
	
		public static Parent getChildInstance() {
			return Child.getInstance();
		}
	
		public static Parent getSecondChildInstance() {
			return SecondChild.getInstance();
		}
	}
	class Child extends Parent {
		private Child() {
		}
	
		public static Child getInstance() {
			return new Child();
		}
	}
	class SecondChild extends Parent {
		private SecondChild() {
		}
	
		public static SecondChild getInstance() {
			return new SecondChild();
		}
	}
  ```
-  입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
  - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
  - JDBC 같은 경우 매개변수에 따라 해당 DB JDBC 객체를 반환한다.

- 정적 팩터리 메서드를 작성하는 시점에는반환할 객체의 클래스가 존재하지 않아도 된다.


##### 단점

- 상속을 하려면 public이나 protected생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
  - 단점이지만 생성자 없이 바로 사용이 가능하므로 장점이 될 수있다.


- 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
  - 생성자처럼 API설명에 명확히 드라나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.
  - 메서드 이름을 널리 알려진 규약을 따라 짓는 식으로 해당 문제를 완화해야 한다.

  - from : 배개 변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 메서드
  ```Date d = Date.from(instant);```

  - of : 여러 개의 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  ```Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);```

  - valueOf : form과 of의 더 자세한 버전
  ```BIgInteger prime = BigINteger.valueOf(Integer.Max_VALUE);```

  - instance or getInstance : 매개변수를 받을 경우 명시한  인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.
  ```StackWalker luke = StackWalker.getInstance(options);```

  - create or newInstance : instance 혹은 getInstance와같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
  ```Object newaRRAY = Array.newInstance(classObject, arrayLen);```

  - getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 팩터리 메서드가 반환할 객체의 타입이다. 
  ```FileStore fs = Files.getFileStore();```

  - newType : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 팩터리 메서드가 반환할 객체의 타입이다. 
  ```BufferedReader br = Files.newBufferReader(path);```

  - type: geyType과 newType의 간결한 버전
  ```List<Complanint> litany = Collection.list(legacyLitany);```

##### 핵심
> 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있으면 고치는게 좋다.
