# Bean Validation

태그: 검증, 스프링 MVC

## Bean Validation

- 검증 기능을 매번 코드로 작성하는 것은 상당히 번거롭다
- 특히 특정 필드에 대한 검증 로직은 대부분 빈 값인지 아닌지, 특정 크기를 넘는지 아닌지와 같이 매우 일반적인 로직이다.
- 이런 검증 로직을 모든 프로젝트에 적용할 수 있게 공통화하고, 표준화 한 것이 Bean Validation이다
    - Bean Validation을 잘 활용하면 애노테이션 하나로 검증 로직을 매우 편리하게 적용할 수 있다
- Bean Validation이란
    - 특정한 구현체가 아니라 Bean Validation 2.0 (JSR-380)이라는 기술 표준이다
    - 검증 애노테이션과 여러 인터페이스의 모음이다
    - Bean Validation을 구현한 기술 중 일반적으로 사용하는 구현체는 하이버네이트 validation이다 (ORM과 관계 없음)

## Bean Validation - 시작

- 의존관계 추가
    - spring-boot-starter-validation 의존관계를 추가하면 라이브러리가 추가된다
- Jakarta Bean Validation
    - jakarta.validation-api : Bean Validation 인터페이스
    - hibernate-validator : 구현체
- 코드
    
    ```java
    public class Item {
    	private Long id;
    
    	@NotBlank
      private String itemName;
    
      @NotNull
      @Range(min = 1000, max = 1000000)
      private Integer price;
    
      @NotNull
      @Max(9999)
    	private Integer quantity;
      public Item() {}
      public Item(String itemName, Integer price, Integer quantity) {
      this.itemName = itemName;
      this.price = price;
      this.quantity = quantity;
    	} 
    }
    ```
    
    - 검증 애노테이션
        - @NotBlank : 빈 값 + 공백만 있는 경우를 허용하지 않는다
        - @NotNull : null을 허용하지 않는다
        - @Range(min = 1000, max = 1000000) : 범위 안의 값이어야 한다
        - @Max(9999) : 최대 9999 까지만 허용한다
    - 참고
        - javax.validation으로 시작하면 특정 구현에 관계없이 제공되는 표준 인터페이스이다
        - org.hibernate.validator로 시작하면 하이버네이트 validator 구현체를 사용할 때만 제공되는 검증 기능이다
    - 동작
        - 검증기 생성
            
            ```java
            ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
            Validator validator = factory.getValidator();
            ```
            
            - 스프링과 통합하면 이런 코드를 직접 사용하지는 않는다
        - 검증 실행
            
            ```java
            Set<ConstraintViolation<Item>> violations = validator.validate(item);
            ```
            
            - 검증 대상을 직접 검증기에 넣고 그 결과를 받는다.
            - Set에는 ConstraintViolation이라는 검증 오류가 담긴다
            - 따라서 결과가 비어있으면 검증 오류가 없는 것이다

## 스프링 통합

- 스프링 부트는 자동으로 글로벌 validator로 등록한다
    
    ```java
    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute Item item, 
    	BindingResult bindingResult, RedirectAttributes redirectAttributes
    ```
    
    - LocalValidatorFactoryBean을 글로벌 Validator로 등록한다
    - 이 Validator는 애노테이션을 보고 검증을 수행한다
    - 이렇게 글로벌 Validator가 적용되어 있기 때문에 @Valid, @Validator만 적용하면 된다
        - @Validated는 스프링 전용 애노테이션이다. 내부에 groups라는 기능을 포함한다
        - @Valid는 자바 표준 검증 애노테이션이다.
    - 검증 오류가 발생하면 FieldError, ObjectError를 생성해서 BindingResult에 넣어준다
- 검증 순서
    1. @ModelAttribute 각각의 필드에 타입 변환 시도
        1. 성공하면 다음으로
        2. 실패하면 typeMismatch로 FieldError 추가
    2. Validator 적용
- 바인딩에 성공한 필드만 Bean Validation 적용
    - BeanValidator는 바인딩에 실패한 필드는 Bean Validation을 적용하지 않는다.
    - 타입 변환에 성공해서 바인딩에 성공한 필드여야 Bean Validation 적용이 의미 있다
    - @ModelAttribute → 각각의 필드 타입 변환 시도 → 변환에 성공한 필드만 Bean Validation 적용

## 에러 코드

- Bean Validation을 적용하고 bindingResult에 등록된 오류 코드는 애노테이션 이름으로 등록된다
- ex) @NotBlank
    - NotBlank.item.itemName
    - NotBlank.itemName
    - NotBlank.java.lang.String
    - NotBlank
- 메시지 등록
    - errors.properties
        
        ```java
        NotBlank={0} 공백X 
        Range={0}, {2} ~ {1} 허용 
        Max={0}, 최대 {1}
        ```
        
- Bean Validation 메시지 찾는 순서
    1. 생성된 메시지 코드 순서대로 messageSource에서 메시지 찾기
    2. 애노테이션의 message 속성 사용 → @NotBlank(message = “공백!”)
    3. 라이브러리가 제공하는 기본 값 사용

## 오브젝트 오류

- 특정 필드가 아닌 오브젝트 관련 오류 (ObjectError)는 어떻게 처리할 수 있을까?
- @ScriptAssert()를 사용하면 된다
    
    ```java
    @Data
    @ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >=
      10000")
    public class Item {
    //...
    }
    ```
    
    - 메시지 코드
        - ScriptAssert.item, ScriptAssert
    - 실제 사용시 제약이 많고 복잡하다.
    - 따라서 오브젝트 오류의 경우 @ScriptAssert를 사용하는 것 보다 오브젝트 오류 관련 부분만 직접 자바 코드로 작성하는 것이 좋다
    

## Bean Validation - 한계

- 데이터를 등록할 때와 수정할 때 요구사항이 다를 수 있다
    - 이 경우 등록과 수정에서 검증 조건의 충돌이 발생하여 같은 Bean Validation을 적용할 수 없을 수 있다.

### groups

- 동일한 객체를 등록할 때와 수정할 때 각각 다르게 검증하는 방법 중 하나이다
    - Bean Validation의 groups라는 기능을 사용하여 등록시 검증할 기능과 수정시 검증할 기능을 각각 그룹으로 나누어 적용할 수 있다.
- groups
    
    
    ```java
    // 저장용 groups
    public interface SaveCheck {
    }
    ```
    
    ```java
    // 수정용 groups
    public interface UpdateCheck {
    }
    ```
    
- Item - groups 적용
    
    ```java
    @Data
    public class Item {
    	@NotNull(groups = UpdateCheck.class) //수정시에만 적용 private Long id;
      @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
      private String itemName;
    
      @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
      @Range(min = 1000, max = 1000000, groups = {SaveCheck.class,
    	UpdateCheck.class})
      private Integer price;
    
      @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    	@Max(value = 9999, groups = SaveCheck.class) //등록시에만 적용 
    	private Integer quantity;
    
    	public Item() {
    	}
    
    	public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
    	  this.quantity = quantity;
    	} 
    }
    ```
    
- 저장 로직에 SaveCheck groups 적용
    
    ```java
    @PostMapping("/add")
    public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item,
    BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    	//...
    }
    ```
    
- groups 기능을 사용해서 등록과 수정시에 각각 다르게 검증할 수 있다
- 하지만 groups 기능을 사용하니 Item과 로직의 전반적인 복잡도가 올라갔다
- 실무에서는 groups 기능보다 주로 등록용 폼과 수정용 폼 객체를 분리해서 사용한다

### Form 전송 객체 분리

- 실무에서는 groups를 잘 사용하지 않는다
    - 등록시 폼에서 전달하는 데이터가 Item 도메인 객체와 딱 맞지 않기 때문이다
    - 그래서 보통 Item을 직접 전달받는 것이 아니라, 복잡한 폼의 데이터를 컨트롤러 까지 별도의 객체를 만들어 전달한다
    - 이것을 통해 컨트롤러에서 폼 데이터를 전달받고, 필요한 데이터를 사용하여 Item을 생성한다
- 폼 데이터 전달에 Item 도메인 객체 사용
    - HTML Form → Item → Controller → Item → Repository
    - 장점
        - Item 도메인 객체를 컨트롤러, 리포지토리 까지 직접 전달해서 중간에 Item을 만드는 과정이 없어 간단한다
    - 단점
        - 간단한 경우에만 적용할 수 있다. 수정시 검증이 중복될 수 있고, groups를 사용해야 한다
- 폼 데이터 전달을 위한 별도의 객체 사용
    - HTML Form → ItemSaveForm → Controller → Item 생성 → Repository
    - 장점
        - 전송하는 폼 데이터가 복잡해도 거기에 맞춘 별도의 폼 객체를 사용해서 데이터를 전달 받을 수 있다.
        - 보통 등록과 수정용으로 별도의 폼 객체를 만들기 때문에 검증이 중복되지 않는다
    - 단점
        - 폼 데이터를 기반으로 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가된다
- 코드
    
    
    ```java
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
    ```
    
    ```java
    @Data
    public class ItemUpdateForm {
      @NotNull
      private Long id;
    
      @NotBlank
      private String itemName;
    
      @NotNull
      @Range(min = 1000, max = 1000000)
      private Integer price;
    }
    ```
    
- 폼 객체를 Item으로 변환하는 과정이 필요하다

## HTTP 메시지 컨버터

- @Valid, @Validated는 HttpMessageConverter에도 적용할 수 있다 (@RequestBody)
- 타입 오류
    - HttpMessageConverter에서 요청 JSON을 ItemSaveForm 객체로 생성하는데 실패한다
    - 이 경우 ItemSaveForm 객체를 만들지 못하기 때문에 컨트롤러 자체가 호출되지 않고 예외가 발생한다
    - Validator도 실행되지 않는다
- 검증 오류
    - 검증에서 오류가 발생하는 경우에는 정상 동작한다
- @ModelAttribute vs @RequestBody
    - @ModelAttribute는 각각의 필드 단위로 세밀하게 적용된다
    - 그래서 특정 필드에 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리 할 수 있다
    - HttpMessageConverter는 각각의 필드 단위가 아니라 전체 객체 단위로 적용된다