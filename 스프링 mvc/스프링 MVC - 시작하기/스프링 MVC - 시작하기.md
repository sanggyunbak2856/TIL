# 스프링 MVC - 시작하기

태그: 스프링 MVC

## 스프링 MVC - 시작하기

- **@RequestMapping**
    - 애노테이션을 활용한 매우 유연하고 실용적인 컨트롤러
    - 스프링에서 가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 RequestMappingHandlerMapping과 RequestMappingHandlerAdapter이다.
    - 실무에서 99.9% 이 방식의 컨트롤러를 사용한다.
- **SpringMemberFormControllerV1 - 회원 등록 폼**
    
    ```java
    @Controller
    public class SpringMemberFormControllerV1 {
        @RequestMapping
        public ModelAndView process() {
            return new ModelAndView("new-form");
        }
    }
    ```
    
    - @Controller
        - 스프링이 자동으로 스프링 빈으로 등록한다 (내부에 @Component 애노테이션이 있어 컴포넌트 스캔의 대상이 된다)
    - @RequestMapping
        - 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다.
        - 애노테이션을 기반으로 동작하므로 메서드의 이름은 임의로 지으면 된다.
    - RequestMappingHandlerMapping은 스프링 빈 중에서 @RequestMapping 또는 @Controller이 클래스 레벨이 붙은 경우에 매핑 정보로 인식한다.

## 스프링 MVC - 컨트롤러 통합

- 컨트롤러 클래스를 유연하게 하나로 통합할 수 있다.
- **SpringMemberControllerV2**
    
    ```java
    @Controller
    @RequestMapping("/springmvc/v2/members")
    public class SpringMemberControllerV2 {
    
        private MemberRepository memberRepository = MemberRepository.getInstance();
    
        @RequestMapping("/new-form")
        public ModelAndView newForm() {
            return new ModelAndView("new-form");
        }
    
        @RequestMapping
        public ModelAndView save() {
            List<Member> members = memberRepository.findAll();
            ModelAndView mv = new ModelAndView("members");
            mv.addObject("members", members);
            return mv;
        }
    
        @RequestMapping("/save")
        public ModelAndView members(HttpServletRequest request, HttpServletResponse response) {
            String username = request.getParameter("username");
            int age = Integer.parseInt(request.getParameter("age"));
    
            Member member = new Member(username, age);
            memberRepository.save(member);
    
            ModelAndView mv = new ModelAndView("save-result");
            mv.addObject("member", member);
            return mv;
        }
    }
    ```
    
    - 이전의 코드는 “/springmvc/v2/members”에 중복이 있었다.
    - 클래스 레벨에 @RequestMapping을 두어 메서드 레벨과 조합한다.
    

## 스프링 MVC - 실용적인 방식

- **SpringMemberControllerV3**
    
    ```java
    @Controller
    @RequestMapping("/springmvc/v3/members")
    public class SpringMemberControllerV3 {
        private MemberRepository memberRepository = MemberRepository.getInstance();
    
        @GetMapping("/new-form")
        public String newForm() {
            return "new-form";
        }
    
        @GetMapping
        public String members(Model model) {
            List<Member> members = memberRepository.findAll();
            model.addAttribute("members", members);
            return "members";
        }
    
        @PostMapping("/save")
        public String save(
                @RequestParam("username") String username,
                @RequestParam("age") int age,
                Model model
                ) {
    
            Member member = new Member(username, age);
            memberRepository.save(member);
    
            model.addAttribute("member", member);
            return "save-result";
        }
    }
    ```
    
    - Model 파라미터
        - Model을 파라미터로 받아 직접 생성하지 않는다.
    - ViewName 반환
        - 뷰의 논리 이름을 반환한다
    - @RequestParam
        - HTTP 요청 파라미터를 @RequestParam으로 받는다.
        - @RequestParam(”username”)은 request.getParameter(”username”)과 거의 같은 코드
    - @GetMapping, @PostMapping
        - @RequestMapping은 URL만 매칭하는 것이 아니라 HTTP Method도 함께 구분할 수 있다
        - 예를들어 /new-form을 GET 방식으로 처리하려면 다음과 같이 처리하면 된다.
            - @RequestMapping(value = ”/new-form”, method = RequestMethod.GET)
        - 이것을 @GetMapping(”/new-form”)으로 사용할 수 있다.
        - Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되어있다.