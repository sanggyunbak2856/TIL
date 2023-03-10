# validator 분리

태그: 스프링 MVC

## Validator 분리 1

- 목표
    - 복잡한 검증 로직을 별도로 분리하자
- 컨트롤러에서 검증 로직이 차지하는 부분은 매우 크다
- 이런 경우 별도의 클래스로 역할을 분리하는 것이 좋다
- 분리한 검증 로직을 재사용할 수 있다
- 스프링은 검증을 체계적으로 제공하기 위해 다음 인터페이스를 제공한다
    
    ```java
    public interface Validator {
        boolean supports(Class<?> clazz);
        void validate(Object target, Errors errors);
    }
    ```
    
    - supports : 해당 검증기를 지원하는 여부 확인
    - validate(Object target, Errors errors) : 검증 대상 객체와 BindingResult
    - 위 인터페이스를 구현한 검증 클래스를 만든다
- ItemValidator
    
    ```java
    @Component
      public class ItemValidator implements Validator {
          @Override
          public boolean supports(Class<?> clazz) {
              return Item.class.isAssignableFrom(clazz);
          }
          @Override
          public void validate(Object target, Errors errors) {
              Item item = (Item) target;
              ValidationUtils.rejectIfEmptyOrWhitespace(errors, "itemName",
    					"required");
    					if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
    					  errors.rejectValue("price", "range", new Object[]{1000, 1000000},
    					}
    					if (item.getQuantity() == null || item.getQuantity() > 10000) {
    					  errors.rejectValue("quantity", "max", new Object[]{9999}, null);
    					}
    					//특정 필드 예외가 아닌 전체 예외
    					if (item.getPrice() != null && item.getQuantity() != null) {
    					  int resultPrice = item.getPrice() * item.getQuantity();
    					  if (resultPrice < 10000) {
    						  errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
    						} 
    					}
    			} 
    }
    ```
    
- ItemValidator 직접 호출하기
    
    ```java
    private final ItemValidator itemValidator;
    @PostMapping("/add")
    public String addItemV5(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    	itemValidator.validate(item, bindingResult);
    	if (bindingResult.hasErrors()) {
              log.info("errors={}", bindingResult);
              return "validation/v2/addForm";
    	}
    	//성공 로직
    	Item savedItem = itemRepository.save(item); 
    	redirectAttributes.addAttribute("itemId", savedItem.getId()); 
    	redirectAttributes.addAttribute("status", true);
    	return "redirect:/validation/v2/items/{itemId}";
    }
    ```
    
    - 검증 부분이 깔끔하게 분리되었다

## Validator 분리 2

- 스프링이 Validator 인터페이스를 별도로 제공하는 이유는 체계적으로 검증 기능을 도입하기 위해서이다
- 검증기를 직접 불러 사용해도 된다
- Validator 인터페이스를 사용해서 검증기를 만들면 스프링의 추가적인 도움을 받을 수 있다
- @InitBinder
    
    ```java
    @InitBinder
    public void init(WebDataBinder dataBinder) {
    	log.info("init binder {}", dataBinder);
    	dataBinder.addValidators(itemValidator);
    }
    ```
    
    - 이렇게 WebDataBinder에 검증기를 추가하면 해당 컨트롤러에서는 검증기를 자동으로 적용할 수 있다
    - @InitBinder → 해당 컨트롤러에만 영향을 준다
- @Validated 적용
    
    ```java
    @PostMapping("/add")
    public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    	if (bindingResult.hasErrors()) {
    	  log.info("errors={}", bindingResult);
        return "validation/v2/addForm";
    	}
    	//성공 로직
    	Item savedItem = itemRepository.save(item); 
    	redirectAttributes.addAttribute("itemId", savedItem.getId()); 
    	redirectAttributes.addAttribute("status", true);
    	return "redirect:/validation/v2/items/{itemId}";
    }
    ```
    
    - validator를 직접 호출하는 부분이 사라지고 대신 검증 대상 앞에 @Validated가 붙었다
    - 동작 방식
        - @Validated는 검증기를 실행하라는 애노테이션이다
        - 이 애노테이션이 붙으면 앞서 WebDataBinder에 등록한 검증기를 찾아서 실행한다
        - 그런데 여러 검증기가 등록된다면 그 중에 어떤 검증기를 실행해야할 지 구분이 필요하다
        - 이 때 supports가 사용된다
        - 여기서는 supports(Item.class)가 호출되고, 결과가 true이므로 ItemValidator의 validate가 호출된다