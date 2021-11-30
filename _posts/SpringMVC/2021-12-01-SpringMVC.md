---
title:  "Spring MVC 패턴"
excerpt: "스프링 MVC 1편 김영한"
categories:
  - Education
tag:
  - Spring
toc: true
---

## MVC 패턴

**서블릿과 JSP의 한계**
- 서블릿으로 개발할 때는 뷰를 위한 HTML을 만드는 작업이 자바 코드와 섞여서 지저분하고 복잡하다.
- JSP 사용함으로 뷰를 생성하는 HTML 작업을 깔끔하게 가져갈 수 있으나 비즈니스 로직과 뷰 영역이 섞이면서 너무 많은 역활을 하게 되었다.

**MVC 패턴의 등장**
- 하나의 서브릿이나 JSP만으로 비즈니스 로직과 뷰 렌더링을 처리함으로써 너무 많은 역할을 맞게 되었다.
- UI를 수정하는 것과 비즈니스 로직을 수정하는 일은 각가 다른 라이프 사이클은데 같이 관리하느 유지보수하기 좋지 않다.
- JSP 같은 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어 이 부분의 기능만 담당하는 것이 가장 효과적이다.
- MVC 패턴은 JSP로 처하는 것을  컨트롤러와 뷰 영역으로 서로 역활을 나는 것을 말하며 웹 애플리케이션은 보통 MVC 패턴을 사용한다.
  * 컨트롤러 : HTTP 요청을 받아 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 모델에 담는다.
  * 모델 : 뷰에 출력할 데이터를 담아둔다. 모델에 담아서 처리하기 때문에 비즈니스 로직이나 데이터 접근을 몰라도 되고, 렌더링에 집중할 수 있다.
  * 뷰 : 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성하는 부분을 말한다.


>__참고__  
>컨트롤러에 비즈니스 로직을 둘 수 있지만 그럴경우 너무 많은역활을 담당하게 된다. 
>그래서 일반적으로 비즈니스 로직은 서비스 계층으 ㄹ별도로 만들어 처리하고 컨트롤러는 서비스를 호출하는 일을 담당한다. 
>비즈니스 로직을 변경하면 비즈니스 로직을 호출하는 컨트롤러의 코드도 변경 될 수 있다.


### 프론트 컨트롤러 패턴
- 프론터 컨트롤러 서블릿 하나로 클라이언트의 요청을 받는다.
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출한다.
- 공통 처리가 가능하고 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 된다.

**스프링 웹 MVC와 프론트 컨트롤러**
- 스프링 웹 MVC의 핵심도 FrontController이다.
- 스프링 웹 MVC의 DispatcherServlet이 FrontController 패턴으로 구현되어 있다.

**어댑터 패턴**
- 프론트 컨트롤러 패턴은 어댑터 패턴을 이용함으로 완전히 다른 컨트롤러들을 처리할 수 있도록 했다.
- 핸들러 어댑터 : 다양한 핸들러(컨트롤러)를 처리할 수 있도록 어댑터 역활을 해줘 다양한 종류의 컨트롤러를 호출할 수 있다.
- 핸들러 : 컨트롤러의 이름을 더 넓은 범위로 확장한 것으로 어댑터가 있기 때문에 컨트롤러의 개념뿐 아닌 어떤 것이든 해당하는 종류의 어댑터만 있으면 다 처리 할 수 있다.


### 스프링 MVC

**스프링 MVC 구조**
![SpringMVC](/assets/images/SpringMVC.GIF)


#### DispatcherServlet
- 부모 클래스에서 HttpServlet을 상속 받아서 사용하고, 서블릿으로 동작한다.
- 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 모든 경로에 대해서 매핑한다.

**요청흐름**
- 서블릿이 호출되면 HttpServlet이 제공하는 service() 호출된다.
- service90를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출된다.

``` java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
	HttpServletRequest processedRequest = request;
	HandlerExecutionChain mappedHandler = null;
	ModelAndView mv = null;
	// 1. 핸들러 조회
	mappedHandler = getHandler(processedRequest);
	if (mappedHandler == null) {
		noHandlerFound(processedRequest, response);
		return;
	}
	// 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
	// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환 mv =
	// ha.handle(processedRequest, response, mappedHandler.getHandler());
	processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}

private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
		HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
	// 뷰 렌더링 호출
	render(mv, request, response);
}

protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
	View view;
	String viewName = mv.getViewName(); // 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
	view = resolveViewName(viewName, mv.getModelInternal(), locale, request); // 8. 뷰 렌더링
	view.render(mv.getModelInternal(), request, response);
}
```

**동작 순서**
1. 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
1. 핸들러 어댑터 조회 : 핸들러를 실행 할 수 있는 핸들러 어댑터를 조회한다.
1. 핸들러 어댑터 실행 : 핸들러 어탭터를 실행한다.
1. 핸들러 실행 : 핸들러 어탭터가 실제 핸들러를 실행한다.
1. ModelAndView 반환 : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 반환해서 반환한다.
1. viewResolver 호출 : 뷰 리졸버를 찾고 실행한다.
1. View 반환 : 뷰 리졸버는 논리 이름을 물리 일므으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
1. 뷰 렌더링 : 뷰를 통해 뷰를 렌더링 한다.

#### 핸들러 매핑과 핸들러 어댑터
핸들러는 애노테이션 기반의 컨틀롤러인 @RequestMapping를 사용할 수 있고 스프링 빈의 이름으로도 할 수 있고 Controller 인터페이스를  상속 받아 사용할 수 있다.  
이렇듯 여러 방식으로 다양한 방식으로 사용을하는데 핸들러 어댑터를 사용함으로써 컨트롤러를 통해 한곳으로 요청을 받아도 모두 처리할 수 있도록 되어 있다.  

**HandlerMapping**
- 0 = RequestMappingHandlerMapping   : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
- 1 = BeanNameUrlHandlerMapping      : 스프링 빈의 이름으로 핸들러를 찾는다.

**HandlerAdapter**
- 0 = RequestMappingHandlerAdapter   : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
- 1 = HttpRequestHandlerAdapter      : HttpRequestHandler 처리
- 2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리


#### HTTP 메시지 컨버터

**Argument Resolver**
컨트롤러의 파라미터는 * @RequestParam, @ModelAttribute, HttpServletRequeste 등 다양한 타입의 객채를 입력하면 제공해준다.  
DispatcherServlet 핸들러 처리를 위해 어댑터를 호출하면 Argument Resolver를 호출하고 Argument Resolver는 컨트롤러가 필요하다고 파라미터에 입력한 객체들을 생성해준다. 이때 Model, HttpServletRequest, @ModelAttrivute등 컨트롤러가 요청한 객체를 생성하여 파라미터로 넘겨준다.  
이렇듯 Argument Resolver는 다양성을 제공해준다.

**ReturnValue Handler**
Argument Resolver가 제공한 파라미터를 사용한 컨트롤러는 결과 값을 반환하는데 뷰 상태경로, ModelAndView, @ResponseBody 등 다양한 반환형식을 지원하는데, 컨트롤러의 반환값은 ReturnValue Handler로 넘겨져 형식에 알맞는 처리를 수행하게 된다.

>Argument Resolver는 컨트롤러가 파라미터에 요청한 다양한 객체들을 생성해서 넘겨주고 ReturnValue Handler는 컨트롤러의 다양한 반환값에 알맞는 처리를 해준다. 둘 모두 핸들러 어댑터와 값을 주고 받는다.
  
요청의경우 왔을때 다양한 방식의 객체를 처리하는 Argument Resolver들이 있는데 이 Argument Resolver들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다.  
  
응답의 경우에도 ReturnValue Handler가 있고 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.

**HTTP 컨버터 위치**
![HTTPConverter](/assets/images/HTTPConverter.GIF)
