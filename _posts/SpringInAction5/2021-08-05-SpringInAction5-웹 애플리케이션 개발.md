---
title:  "Spring-웹 애플리케이션 개발"
excerpt: "스프링 인 액션 5"
categories:
  - Study
tag:
  - Spring
  - SpringInAction
toc: true
---

##### 도메인 설정하기
애플리케이션의 도메인은 해당 애플리케이션의 이해에 필요한 개념을 다루는 영역이다. 
-  DB layer와 View Layer에 각각 사용하는 도메인 정보 확인 필요..

Lombok
- Lombok 라이브러리로 도메인 클래스에 @Data 애노테이션을 지정하면 final 속성들을 초기화 하는 생성자는 물론이고 속성들의 게터와 세터를 런타임시에 자동으로 생성해준다.
- 개발시점에는 에러가 나타나는데 STS의 확장으로 Lombok을 추가하면 코드 작성 시점에서도 속성 관련 메서드들이 자동 생성되어 에러가 나타나지 않는다.

유호성 검사 API
- 유호성을 검사할 클래스에 검사 규칙을 선언한다.
- 유호성 검사를 해야 하는 컨트롤러 메서드에 검사를 수행한다는 것을 지정한다.
검사 에러를 보여주도록 폼 뷰를 수정한다.


##### 컨트롤러 클래스 생성
컨트롤러는 스프링 MVC 프레임워크의 중심적인 역활을 수행한다. HTTP 요청을 처리하고, 브라우저에 보여줄 HTML을 뷰에 요청하거나, 또는 REST 형태의 응닫 몸체에 직접 데이터를 추가한다.

@Slf4J 
- 해당 애노테이션은 컴파일시 Lombok에 제공되며, 이 클래스에 자동으로 SLF4J Logger를 생성한다.

@Controller
- 지정한 클래스가 컨트롤러로 식별되게 하며, 컴포넌트 검색을 해야 한다는 것을 나타낸다 스프링이 해당 클래스를 찾은 후 스프링 애플리케이션 컨텍스트의 빈으로 이 클래스의 인스턴스를 자동 생성한다.

@RequestMapping
- 클래스 수준으로 적용될 때는 해당 컨트롤러가 처리하는 요청의 종류를 나타낸다. 지정한 패턴으로 시작하는 요청을 처리


스프링 MVC 요청-대응 애노테이션
- @RequestMapping : 다목적 요청을 처리한다.
- @GetMapping : HTTP GET 요청을 처리한다.
- @PostMapping : HTTP Post 요청을 처리한다.
- @PutMapping : HTTP PUT 요청을 처리한다.
- @DeleteMapping : HTTP DELETE 요청을 처리한다.
- @PatchMapping : HTTP PATCH 요청을 처리한다.

Model 객체의 속성
- Model은 컨트롤러와 데이터를 보여주는 뷰 사이에서 데이터를 운반하는 객체이다. 궁극적으로 Model 객체의 속성에 있는 데이터는 뷰가 알 수 있는 서블릿 요청 속성들로 복사된다.



##### 자주 사용하는 Thymeleaf
th:text
- 화면에 값을 표시
- <span th:text="${}"></span>

th:if
- 조건문
- <div th:if="${}"></div>

th:errors
- value에 error가 있는경우
<li th:errors="*{id}" />

th:action
form 태그에서 URL 보낼때 사용

th:object
form 태그에서 submit할때 설정한 객체로 받아짐, Taco 객체일때 taco 소문자로 설정해도 맵핑 가능함

th:field
각 필드에 매핑해준다.


##### JDBC
- JdbcTemplate 
  - 클래스기반으로 JDBC를 사용할떄 요구되는 모든 형식적이고 상투적인코드 없이 개발자가 관계형 데이터벵스에 대한 SQL 연산을수행할수 있는 방법을 제공한다.
- JDBC 리퍼지터리
  - 데이터 처리에 대한 연산을 정하고 인터페이스를 만들어 메서드로 정의한다. data 관련 처리 경로는 src/main/java/../data에 지정
  - Repository 인터페이스를 구현한다.
    - @Repository 애노테이션을지정하여 스플이컴포넌트 검색에서 이 클래스를 자동으로 찾아서 스프링 애플리케이션 컨텍스트의 빈으로 생성되도록한다.
    - @Autowired 애노테이션을 통해서 스프링이 해당 빈을 JdbcTemplate에 주입되도록 한다.
    - 생성자에서 JdbcTemplate참조를 인스턴스 변수에 저장한다.
    - 처음 정의 한 인터페이스를 implement하여 인터페이스를 구현한다.
- 스키마 정의 및 데이터
  - schema.sql이라는이름의 파일이 애플케이션 classpath의 루트경로에 있으면 애플리케이션이 시작될 때 schema.sql 파일의 SQL이사용 중인 데이터베이스에서 자동 실행된다. /src/main/resources
  - 스프링 부트는 애플리케이션이시작될 때 data.sql이라는 이름의 파일도 실행되도록 한다. 위치 똑같음
- 데이터베이스 생성 ID
  - 연관된 테이블에 데이터가생성되었을떄 생성되는 ID를알아야 한다. 그래야 ID값으로 데이터를 참조할 수있다.
  - update() 메서드로는 ID를 얻을 수 없아 PreparedStatementCreator 객체와 KeyHolder 개체를 인자로 받는다.
  - 생성된 데이터의 ID를 제공하는 것이 KeyHolder이다.그러나 이것을 사용하기 위해선 PreparedStatementCreator 생성해야 한다
  ``` java
  private long saveTacoInfo(Taco taco) {
		taco.setCreateAt(new Date());
		PreparedStatementCreator psc =
				new PreparedStatementCreatorFactory(
						"insert into Taco (name, createdAt) values (?, ?)",
						Types.VARCHAR, Types.TIMESTAMP
						).newPreparedStatementCreator(
								Arrays.asList(
										taco.getName(),
										new Timestamp(taco.getCreateAt().getTime())));
		
		KeyHolder keyHolder = new GeneratedKeyHolder();
		jdbc.update(psc, keyHolder);
		return keyHolder.getKey().longValue();
	}
  ```
- @Modelattribute
  - 객체가모델에 생성되도록 해준다.
  - 매개변수에 애노테이션이 지정되어 잇으면 이 매개변수의 값이 모델로부터 전달되어야 한다는 것과 스프링 MVC가 이 매개변수에 요청 매개변수를 바인딩하지 않아야 한다는 것을 나타내기 위해서다.
  - 하나의 세션에서 생성되는게 아닌 다수의 HTTP 요청에 걸쳐서 존재해야 하는 경우 클래스 수준의 @SessionAttributes 애노테이션을 같은 모델 객체에 지정하면 세션에서 계속 보존되면서 다수의 요청에 걸처 사용될 수 있다.  
