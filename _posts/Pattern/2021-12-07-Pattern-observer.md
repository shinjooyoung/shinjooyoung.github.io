---
title:  "observer"
excerpt: "디자인 패턴"
categories:
  - Study
tag:
  - Java
  - Pattern
toc: true
---

## 옵서버 패턴
- 데이터의 변경이 발생했을 경우 상대 클래스나 객체에 의존하지 않으면서 데이터 변경을 통보하고자 할 때 유용하다.
- 새로운 파일이 추가되거나 기존 파일이 삭제되었을 떄 탐색기는 이를 즉시 표시할 필요가 있다.
- 파일이 추가되가나 삭제되었을떄 탐색기는 직접 의존하지 않는 방식으로 설계되어야 한다.

>옵서버 패턴은 통보대상 객체의 관리를 Subject 클래스와 Observer 인터페이스로 일반화 한다.
> 그러면 데이터 변경을 통보한느 클래스(ConcreateSubject)는 통보 대상 클래스나 객체(ConcreateObserver)에 
> 대한 의존성을 없앨 수 있다. 결과적으로 옵서버 패턴은 통보 대상 클래스나 대상 객체의 변경에도 ConcreateSubjet 클래스를 수정 없이 그대로 사용할 수 있도록 한다.


### 패턴 수행 작업

**Observer**
- 데이터의 변경을 통보 받는 인터페이스 Subject에서 Observer 인터페이스의 update 메서드를 호출함으로써 ConcreateSubject 데이터 변경을 ConcreateObserver에게 통보한다.
- 해당 인터페이를 구현한 객체는 변경 관리 대상이 변경되었을떄 ConcreateSubject에 의해 update가 실행된다.
- 자바에서 Observer 인터페이스를 지원한다.

**Subject**
- ConcreateObserver 객체를 관리하는 요소
- Observer 인터페이스를 참조해서 ConcreateObserver를관리하므로 ConcreateObserver의 변화에 독립적일 수 있다.
- Observer의 인터페이스만 알고 있으니 ConcreateObserver가 변화를 하든 안하든 update를 실행시켜 준다.
- 자바에서 Obserable 클래스를 지원한다.

**ConcreateSubject**
- 변경 관리 대상이 되는 데이터가 있는 클래스
- 데이터 변경을 위한 메서드인 setState가 있으며 setState에서는자신의 데이터인 subjectState를 변경하고 Subject의 notify를 호출하여 객체 변경을 통보한다.
- 변화가 생기면 Observer의 구현체들을 update를 호출하게 하여 변화를 알려준다.

**ConcreateObserver**
- ConcreateSubject의 변경을 통보받는 클래스
- Observer 인터페이스의 update 메서드를 구현함으로써 변경을 통보 받는다.
- 변경된 데이터는 ConcreateSubject의 getState메서드를 호출 함으로써 변경을 조회한다.

> 변경관리 대상 객체와 변화를 구독하는 옵서버를 추상화 함으로써 서로의 구현체가 변경이되어도 기능이 가능하며
> 변경관리 대상 객체의 변화없이 옵서버를 Implement받아 확장이 가능하다.
