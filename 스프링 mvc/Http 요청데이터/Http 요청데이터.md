# Http 요청데이터

태그: 스프링 MVC

## HttpServletRequest

- 서블릿은 개발자 대신 Http 요청 메시지를 파싱하고, 그 결과를 HttpServletRequest 객체에 담아서 제공한다
- HttpServletRequest를 사용하면 다음과 같은 요청 메시지를 편하게 조회할 수 있다
    - START LINE
        - 메서드, URL, 쿼리 스트링, 스키마, 프로토콜
    - 헤더
    - 바디
        - form 파라미터, 메시지 바디 데이터 직접 조회
- 부가기능
    - 임시 저장소 기능
        - 해당 HTTP 요청이 시작할 때 부터 끝날 때 까지 유지되는 임시 저장소 기능
        - 저장 : request.setAttribute(name, value)
        - 조회 : request.getAttribute(name)
    - 세션 관리 기능
        - request.getSession(create: true)

## HTTP 요청 데이터

- **GET - 쿼리 파라미터**
    - /url?username=hello&age=20
    - 메시지 바디 없이 url의 쿼리 파라미터에 데이터를 포함해서 전달
    - 검색, 필터, 페이징에서 많이 사용됨
    - HttpServletRequest 쿼리 파라미터 조회
        
        ```java
        String username = request.getParameter("username"); // 단일 파라미터 조회
        Enumeration<String> parameterNames = request.getParameterNames(); // 파라미터 이름들 조회
        Map<String, String[]> parameterMap = request.getParameterMap(); // 파라미터를 Map으로 조회
        String[] username = request.getParameterValues("username"); // 복수 파라미터 조회
        // 복수 파라미터에서 단일 파라미터 조회 -> request.getParameterValues()의 첫번째 값을 반환
        ```
        
- **POST - HTML form**
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달
        - username=hello&age=20
    - 회원가입, 상품 주문 등에서 사용하는 방식
    - application/x-www-form-urlencoded 형식은 쿼리 파라미터 형식과 같다
        - 쿼리 파라미터 조회 메서드를 그대로 사용하면 된다
    - HTML Form 형식으로 데이터를 전달하면 HTTP 메시지 바디에 해당 데이터를 포함해서 보내기 때문에 바디에 포함된 데이터가 어떤 형식인지 content-type을 꼭 지정해야한다.
- **HTTP 메시지 바디에 데이터를 직접 담아서 요청**
    - HTTP API에서 주로 사용 (JSON, XML, TEXT)
    - 데이터 형식은 주로 JSON 사용
    - POST, PUT, PATCH
    - JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 Jackson, Gson 같은 JSON 변환 라이브러리를 추가해서 사용해야 한다.
        - 스프링 부트는 기본적으로 Jackson 라이브러리를 함께 제공한다