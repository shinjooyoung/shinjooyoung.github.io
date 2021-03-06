---
title:  "Spring-데이터로 작업하기"
excerpt: "스프링 인 액션 5"
categories:
  - Study
tag:
  - Spring
  - SpringInAction
toc: true
---

###### 폼 입력 유효성 검사
- 스프링은 자바의 빈 유효성 검사 API를 지원한다. 이것을사용하면 애플리케이션에 추가 코드를 작성하지않고 유효성 검사 규칙을 쉽게 선언할 수 있다.
- 스프링 MVC에 유효성 검사 적용방법
  - 유효성음 검사할클래스에 검사 규칙을선언
  - 유효성 검사를해야 하는컨트롤러 메서드에 검사를수행한다는 것을 지정
  - 검사 에러를 보여주도록 폼 뷰를 수정
- 유효성 검사 규칙선언
  - @NotNull : null 확인
  - @Size : 입력값의 길이 지정, min, max
  - @NotBlank : 입력되지 않았는지 확인 
  - @CreditCardNumber : 속성 값이 Luhn 알고리즘 검사에 합격한 유효한 신용 카드 번호이어야 한다는 것을 선언하는 것이다. 그러나 실제 존재하는 것이지, 대금 지불에 사용될 수있는지까지는 검사가 되지 않는다.
- 유효성 검사 적용
  - 컨트롤러에서 검사 규칙이 선언된 파라미터 값 앞에 @Valid 애노테이션을 붙여주고 에러 상세 내역을 바들 수 있도록 Errors errors 를추가한다.
  - @Valid 애노테이션은 제출된 객체의 유효성 검사를 수행 하라고 스프링 MVC에 알려주고 에러가 있는 경우 Errors 객체에 저장된다. hasErrors() 메서드를 호출하여 검사 에러가 있는지확인한다.
- 에러 표시
  - Thymleaf는 fields와 th:errors 속성을 통해 Error 객체의 편리한 사용 방법을 제공한다.
  - th:if 속성은 보여줄지 말지를 결정하며 th:errors 속성은 지정한 필드를 참조하여 에러발생시 지정된 메시지를 검사 에러 메시지로 교체한다.
 
###### 뷰 컨트롤러 작업
- 모델 데이터나 사용자 입력을 처리하지 않는 간단한 컨트롤러의 경우 뷰의 요청을 절달하는일만하느 컨트롤러(뷰 컨트롤러)를 선안할 있다.
- webConfig
  - 뷰 컨트롤러의 역활을 수행하는구성 클래스이며, 가장 중요한 것은 WebMvcConfigurer 인터페이슬르 구현한다는것이다.
  - WebMvcConfigurer 인터페이스는 스프링 MVC를 구성하는 메서드를 정의하고 있다. 그리고 인터페이스임에 불구하고, 정의된 모든 머ㅔ서드의 기본적인구현을 제공한다. 따라서 필요한 메서드만 선택해서 오버라이딩 하면된다.
  - addViewControllers() : 하나 이상의 뷰 컨트롤러를 등록하기위해 사용될 수 잇는 ViewControlerRegistry를 인자로 받는다. 뷰 컨트롤러가 GET 요청을 처리하는 경로인 "/"를 인자로 전다하여 addViewController()를 호출한다. "/" 경로의 요청이 전달되어야 하는 뷰로 home을 지정하기 위해 setViewName()을 호출한다.
  
###### 스프링 지원 뷰 템플릿
- FreeMarker
- Grooy 템플릿
- JSP
  - jsp를 사용하는 경우 자바 서블릿 컨테이너는 /WEB-INF 밑에서 코드를 찾는다. 애플리케이션을 실행 가능한 JAR 파일로 생성한다면 요구사항을 충족 시킬수 없어 WAR 파일로 생성하고 종전의 서브릿컨테이너에 설치하는 경우에는 jsp를 선택해야 한다.
- Mustache
- Thymelaf

###### 템플릿 캐싱
- 템플릿은 최초 사용될 때 한번만  파싱된다. 그리고 파시된 결과는 향 후 사용을 위해 캐시에 저장된다. 템플릿 캐싱으로 인해 화면을 수정해도 이전과 동일하게 표시가 되는데 템플릿 캐싱을 false로 설정하면 해당 기능을 사용하지 않을수 있다. 실제 배포시에는 다시 true로 변경해야 한다. 
 
###### 데이터터 저장
- SimpleJdbcInsert 
  - 데이터를더 쉽게 테이블에추기하기위해 JdbcTemplate를 래핑한 객체다.
- SessnoStatus
  - 데이터 객체가 데이터베이스에 저자된 후에는 세션 보존할 필요가 없어 컨트롤로 메소드에서 인자로 받아 setComplete() 메서드를 호출하여 세션을 재설정 한다.
  - 세션을 재설정 하지 않으면 이전에 저장된 세션을 계속 가지고 있을수 있다. 

###### JPA

>스프링 데이터에서는 리퍼지터리 인터페이스를 기반으로 이 인터페이스를 구현하는 리퍼지터리를 자동 생성해 준다.

- 스프링 데이터 JPA : 관계형 데이터베이스의 JAP 퍼시스턴스
- 스프링 데이터 MongoDB : 몽고 문서형 데이터베이스의 퍼시스턴스
- 스프링 데이터 Neo4: Neo4j 그래프 데이터베이스의 퍼시스턴스
- 스프링 데이터 레디스 : 레디스 키-값 스토어의 퍼시스턴스
- 스프링 데이터 카산드라 : 카산드라 데이터베이스의 퍼시스턴스

1. 도메인 객체 에노테이션
  - @Entity : 클래스 수준에 추가하며 JPA 개체로 선언한다.(javax.persistence.Entity)
  - @id : 속성에 추가하며 데이터베이스의 개체를 고유하게 실벽한다는 것을 나타낸다.
  - @NoArgsConstructor : JPA에는 인자 없는 생성자를 가져야 되기 때문에 Lombok의 에너테이션을 지정한 것이다.
    - 인자 없는 생성자 사용을 원치 않으면 access 속성을 AccessLevel.PRIVATE로 설정여 외부에서 사용하지 못하게 한다.
    - 초기화가 필요한 final이 있는 경우 force를 true로 설정한다.
  - @RequiredArgsConstructor : @Data는 인자가 잇는 생성자를 자동으로 추가한다. @NoArgsConstructor와 겹치는 경우 사용하면 동시에 사용 가능하다.
  - @GeneratedValue : id 속성들은 데이터베이스가 자동으로 새엉해 주는 ID값을 사용하기 위해 strategy를 GenerationType.AUTO로 설정해서 사용한다.
  - @ManyToMany : 관계를 선언하기 위해 사용, 하나의 속성이 여러 객체에 포함될수 있을때 사용
  - @PrePersist : 메서드에 지정하며 객체가 저장되기 전에 속성에 현재 일자와 시간으로 설정해준다.
  - @Table : 지정한 name의 데이터베이스 테이블에 데이터가 저장된다. 지정하지 않는 경우 객체의 이름으로 데이터를 저장하므로 객체명이 DB의 예약어등 사용하지 못할때 지정한다.
1. JPA 리퍼지터리
  - CrudRepostory : 스프링 데이터에서는 CrudRepostory 인터페이스를 확장 하여 CRUD 연산을 위한 메서드들을 사용할 수 있다.
    - 매개변수화 타입으로 첫 번째 매개변수는 리퍼지터리에 저장되는 개체 타입이며, 두 번째는 개체 ID 속성 타입이다. 
    - 구현하는 클래스를 작성하지 않아도 애플리케이션이 시작될 떄 스프링데이터 JPA가 각 구현체를 자동으로 생성해준다.
1. JPA 리퍼지터리 커스터 마이징
  - 기본적인 CRUD가 아닌 다른 기능들은 메서드를 선언하면 사용할 수 있다.
  - 스프링은 리퍼지터리 구현체를 생성할때 메서드의 이름을 분석하여 용도가 무엇인지 파악한다.
  - 간단한 메서드 이름뿐 아니라 복잡한 이름도 처리할수 있다.
    - 동사, 생략가능한 처리대상, By 단어 서술어로 구성된다. 
    ``` java
    List<Order> readOrdersByDeliveryZipAndPlacedAtBetween(
    		String deliveryZip, Date startDate, Date endDate);
    ``` 
    ![usecase](/assets/images/sql.GIF)
    
  - 지정된 열의 값을 기준으로 결과를 정렬하기 위해 메서드 이름의 끝에 OrderBy를 추가할 수 있다.
  - 복잡한 쿼리의 경우는 메서드 이름으로만 감당하기 어려워 어떤이름이든 원하는 이름을 지정후 @Query애노테이션을 지정한다.
  - 이름 규칙을 준수하여 수행하는 것이 불가능할때도 @Query를 사용할수있다.
  ``` java
  // 시애틀에 배달된 모든주문 요청
  @Query("Order o where o.deliveryCity='Seattle'")
  List<Order> readOrdersDeliveredInSeattle();
  ```