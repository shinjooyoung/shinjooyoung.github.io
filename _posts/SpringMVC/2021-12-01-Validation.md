---
title:  "Validation"
excerpt: "스프링 MVC 2편 김영한"
categories:
  - Education
tag:
  - Spring
toc: true
---

## 검증
컨트롤러의 중요한 역할중 하나는 HTTP 요청이 정상인지 검증하는 것이다.  
정상 로직보다 이런 검증 로직을 잘 개발하는 것이 더 어려울 수 있다.

**클라이언트 검증, 서버 검증**
- 클라이언트 검증은 조작할 수 있어 보안에 취약하다.
- 서버만으로 검증하면, 즉각적은 고객 사용성이 부족하다.
- 두개를 적절히 섞어 사용하되, 최족적으로 서버 검증은 필수이다.
- API 방식을 사용하면 스펙을 잘 저으이해서 검증 오류를 API 응답 결과에 잘 남겨주어야 한다.

**스프링 MVC의 Bean Validatgor를 사용 방법**  
스프링 부트가 spring-boot-starter-validation 라이브러리를 넣으면 자동으로 Bean Validator를 인지하고 스프링에 통합한다.

**스프링 부트는 자동으로 글로벌 Validator로 등록한다.**  
LocalValidatorFactoryBean을 글로벌 Validator로 등록한다 이 Validator는 @NotNull 같은 애노테이션을 보고 검증을 수행한다. 이렇게 글로벌 Validator가 적용되어 있기 때문에, @Valid, @Validated만 적용하면 된다. 검증 오류가 발생하면, FieldError, ObjectError를 생성해서 BindingResult에 담아준다.

### 검증 순서
1. ModelAttrivute 각각의 필드에 타입 변환 시도
  - 성공하면 다음으로
  - 실패하면 typeMismatch로 FieldError 추가
1. Validator 적용

**바인딩에 성공한 필드만 Validation 적용**
BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다.  
타입 변환에 성공해서 바인딩에 성공한 필드여야 BeanValidation 적용이 의미 있다.  
(일단 모델 객체에 바인딩 받는 값이 정상으로 들어와야 검증도 의미가 있다.)

@ModelAttrivute -> rkrrkrdml vlfem xkdlq qusghkstleh -> 벼ㅓㄴ환에 성공한 필드만 Bean Validation 적용

## Bean Validation - group
동일한 모델 객체들 등록할 떄와 수정할 떄 각각 다르게 검증이 가능하다.

**검증 방법**
- Bean Validation의 group 기능을 사용한다.
- Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용한다.

### Bean Validation group
등록시에 검증할 기능과 수정시에 검증할 기능을 각각 그룹으로 나누어 적용할 수 있다.

**gorus 적용 방법**  
Save, Update 두가지의 그룹을 나눈다고 하면 Save, Update 2개의 체크용 인터페이스를 만들어 도메인 객체의 필드의 Annotation에 그룹을 지정한다.  
실제 검증시 적용하려면 컨트롤러에 Validated Annotation에 인터페이스를 지정하여 해당 인터페이스가 그룹으로 지정된것들만 검증하도록 한다.

``` java
 @NotNull(groups = UpdateCheck.class) // 수정 요구사항

    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
    private String itemName;

    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
    private Integer price;


    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Max(value = 9999, groups = {SaveCheck.class}) 
    private Integer quantity;
    
   @PostMapping("/add")
  public String addItem2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

      //특정 필드가 아닌 복합 룰 검증
      if (item.getPrice() != null && item.getQuantity() != null) {
          int resultPrice = item.getPrice() * item.getQuantity();
          if (resultPrice < 10000) {
              bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
          }
      }
    }
    
```
실무에서는 groups를 잘 사용하지 않는데 그 이유는 등록용 폼 객체와 수정요 폼객체를 분리해서 사용하기 때문이다.

### From 전송 객체 분리
실무에서는 등록과 수정시 관련된 데이터만 받는게 아닌 약관정보 등 추가적으로 많은 데이터가 넘어온다. 그래서 보통 복잡한 폼의 데이터를 컨트롤러까지 전달할 별도의 객체를 만들어서 전달한다. 예를들어 ItemSaveForm이라는 폼을 전달받는 전용 객체를 만들어서 @ModelAttrivute로 사용한다. 이것을 통해 컨트롤러에서 폼 데이터를 전달 받고, 이후 컨트롤러에서 필요한 데이터를 사용해서 Item을 생성한다.

**폼 데이터 전달에 Item 도메인 객체 사용**  
HTML Form -> Item -> Controller -> Item -> Repository  
- 장점 : Item 도메인 객체를 컨트롤러, 리포지토리 까지 직접 전달해서 중간에 Item을 만드는 과정이 없어서 간단하다.
- 단점 : 간단한 경우에만 적용할 수 있다. 수정시 검증이 중복될 수 있고, groups를 사용해야 한다.

**폼 데이터 전달을 위한 별도의 객체 사용**
HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository  
- 장점 : 전송하는 폼 데이터가 복잡해도 거기에 맞춘 폼 객체르 ㄹ사용해서 데이터를 전달받을 수 있다. 등록과 수정용으로 별도의 폼 객체를 만들기 떄문에 검증이 중복되지 않는다.
- 단점 : 폼 데이터를 기반으로 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가된다.
  
등록과 수정은 오나전히 다른데이터가 넘어온다. 그리고 검증 로직도 많이 달라진다. 그래서 ItemUdpateForm 이라는 별도의 객체로 데이터를 전달받는 것이 좋다.

Item 도메인 객체를 폼 전달 데이터로 사용하고 그대로 넘기면 편하지만 무수히 많은 데이터가 넘어온다 그리고 Item을 생성하는데 필요한 추가 데이터를 데이터베이스나 다른 곳에서 찾아와야 할 수도 있다.

따라서 폼 데이터 전달을 위한 별도의 객체를 사용하고 등록, 수정용 폼 객체를 나누면 등록, 수정이 완전히 분리되기 떄문에 groups를 적용할 일은 드물다.

>이름은 의미 있게 지음녀 되고, 일관성이 있어야 한다.  
>ItemSaveForm, ItemSaveRequest, ItemSaveDto 등..

``` java
@Data
public class ItemSaveForm {

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;
}

@Data
public class ItemUpdateForm {

    @NotNull
    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    //수정에서는 수량은 자유롭게 변경할 수 있다.
    private Integer quantity;
}


@PostMapping("/add")
    public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    ...
}

 @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @Validated @ModelAttribute("item") ItemUpdateForm form, BindingResult bindingResult) {
}

```


## Bean Validation - HTTTP 메시지 컨버터
@Valid, @Validated는 HttpMessageConverter (@RequestBody)에도 적용할 수 있다.

>**참고**  
>@ModelAttrivute는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰때 사용한다.  
>@RequestBody는 HTTP Body의 데이터를 객체로 변환할 때 사용한다. 주로 API JSON요청을 다룰 때 사용한다.

**AP의 경우 3가지 경우를 나누어 생각해야 한다.**
- 성공요청 : 성공
- 실패요청 : JSON을 객체로 생성하는 것 자체가 실패함
- 검증 오류 요청 : JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함

**@ModelAttrivute VS @RequestBody**  
HTTP 요청 파라미터를 처리하는 @ModelAttrivute는 각각의 필드 단위로 세밀하게 적용된다. 그래서 특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있었다.  
HTTPMessageConverter는 전체 객체 단위로 적용된다. 따라서 메시지 컨버터의 작동이 성공해서 Item객체르 만들어야 @Valid, @Validated가 적용된다.

- @ModelAttrivute 는 필드 단위로 정교하게 바인딩 된다. 특정 필드가 바인딩 되지 않아도 나머지 필드는 정상 바인딩되고, Validator를 사용한 검증도 적용할 수 있다.
- @RequestBody는 HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자체가 진행않고 예외가 발생한다. 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.


### 정리



