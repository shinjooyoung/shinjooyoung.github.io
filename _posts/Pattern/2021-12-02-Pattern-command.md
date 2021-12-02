---
title:  "command"
excerpt: "디자인 패턴"
categories:
  - Study
tag:
  - Java
  - Pattern
toc: true
---

## 커맨드 패턴
- 이벤트가 발생했을때 실행될 기능이 다양하면서도 변경이 필요한 경우에 이벤트를 발생시키는 클래스를 변경하지 않고 재사용하고자 할떄 유용하다.
- 실행될 기능을 캡슐화함으로써 기능의 실행을 요구하는 호출자 클래스와 기능을 실행하는 수신자 클래스 사이의 의존성을 제거한다.
- 실행될 기능의 변경에도 호출자 크래스를 수정 없이 그대로 사용할 수 있도록 해준다.


### 패턴 수행 작업

**Command**
- 실행될 기능에 대한 인터페이스, 실행될 기능을 excute 메서드로 선언함


**ConcreateCommand**
- 실제로 실행되는 기능을 구현
- Command라는 인터페이스를 구현한 구현체로 Receiver의 기능을 실제로 수행한다.
 
**Invoker**
- 기능의 실행을 요청하는호출자 클래스

**Receiver**
- ConcreteCommand에서 execute 메서드를 구현할 때 필요한 클래스
- ConcreateCommand의 기능을 실행하기 위해 사용하는 수신자 클래스


### 전략 패턴과 커맨더 패턴의 차이

커맨드 패턴이 인터페이스를 구현하여 사용하다 보니 전략 패턴과 비슷한 느낌을 많이 받았다.  
맞는지는 잘 모르겠으나 개인적인 생각인 두개의 차이점을 한번 써보려고 한다.  
  
전략 패턴은 무엇을 할지는 이미 정해진 상태이다. 가령 서울에서 부산을 간다고 한다면 다양한 방법으로 이동이 가능하다.
Mover라는 인터페이스를 만들고 move라는 추상메소드를 정의해 이동 수단 객체들이 상속 받도록 한다.  
비행기, 자동차, 기차, 자전거 등의 객체가  상속 받았으며 각자의 방식대로 move메소드를 구현한다.
그렇다면 서울에서 부산을 이동하는 객체는 구현체의 기능을 직접 이용하는게 아닌 Mover를 통해 다양한  
방법으로 이동이 가능하다.  
  
이와 같이 무엇을 할지 정해진 상태에서 어떻게 할지를 정해야 할때 전략패튼을 사용하게 된다.  
여기서 중요한점이라고 생각되는 부부은 커맨더 패턴은 클라이언트에게 "무엇"을 할지 요청 자체를 캡슐화 하는 것이라면  
전략 패턴은 객체 자신이 "어떻게" 할지 행동을 캡슐화 하는 부분이다. 
  
위의 전략패턴을 사용한 서울에서 부산을 가는 방식은 어떻게 보면 커맨드 패턴으로도 사용이 가능하다.
Command 인터페이스를 만들고 비행기, 자동차, 기차등은 reciver가 되며 이를 실행시키는  
ConcreateonCommand을 만들면 된다. 그럼 요청이 왔을때 invoker에서 실행을 하면된다.
  
이처럼 사용이 가능해 보이나 전략 패턴에서 말했듯  커맨더 패턴은 "어떻게" 할지가 아닌 "무엇"을 할지가 중요하다.
무엇이라는 관점으로 생각해보면  부산을 갈때 이동수단을 선택하는 부분은 해당 패턴과 맞지 않다.  
   
커맨더 패턴을 사용한다고 하면 식당 상황에서 사용할 수 있다.  

주문자는 Client, 점원은 Invoke, 주문 ConcreateCommand, 주방장은 Receiver이다.  
주문자는 주문을 점원에게 말한다. 그러면 점원은 주문을 주방장에게 만들도록 요청한다.
이를 패턴으로 본다고 하면 client(주문자)는 ConcreateCommand(주문)을 생성하고 Invoker(점원)에게 setCommand()를 한다.
그럼  Invoke(점원)는  ConcreateCommand(주문)를 호출하여  Receiver(주방장)의 기능이 실행된다.  


1. Client -(create)-> ConcreateCommand 
  * ConcreateCommand의 excute() 메서드 안에 Reciver의 메서드가 호출되어 기능 실행이 가능함
  * Receiver receiver = new Receiver();
  * Command concreateCommand = new ConcreateCommand(receiver);
1. Client -(setCommand)-> Invoke
  * 기능을 실행을 요청하는 호출자 클래스에게 기능을 넣어줌
  * Invoker invoke = new Invoker(concreateCommand);
1. Invoker -(excute)-> concreateCommand
  * 호출자는 ConcreateComman의 기능을 실행
  * ConcreateComman에 의해 Receiver의 기능이 작동함