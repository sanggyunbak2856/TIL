# 서블릿, JSP, MVC 패턴

태그: 스프링 MVC

## 서블릿으로 회원 등록 페이지 만들기

- 서블릿으로 회원 등록 HTML 폼 제공
    
    ```java
    @Override
    protected void service(HttpServletRequest request, HttpServletResponseresponse) throws ServletException, IOException {
          response.setContentType("text/html");
          response.setCharacterEncoding("utf-8");
    			PrintWriter w = response.getWriter();
    			w.write("<!DOCTYPE html>\n" +
    			"<html>\n" +
    			"<head>\n" +
    			"    <meta charset=\"UTF-8\">\n" +
    			"    <title>Title</title>\n" +
    			"</head>\n" +
    			"<body>\n" +
    			"<form action=\"/servlet/members/save\" method=\"post\">\n" +
    			"    username: <input type=\"text\" name=\"username\" />\n" +
    			"    age:      <input type=\"text\" name=\"age\" />\n" +
    			" <button type=\"submit\">전송</button>\n" + "</form>\n" +
    			"</body>\n" +
    			"</html>\n");
    }
    ```
    
- 위와 같이 자바 코드로 HTML을 제공해야 하므로 쉽지 않다
- **템플릿 엔진**
    - 서블릿 덕분에 동적으로 원하는 HTML을 만들 수 있다
    - 하지만 자바 코드로 HTML을 만들어 내는 것은 매우 복잡하고 비효율적이다
    - HTML 문서에 동적으로 변경하는 부분을 자바로 변경하는 것이 더 쉬울 것이다
        
        → 템플릿 엔진이 나온 이유
        
        → 템플릿 엔진을 사용하면 HTML 문서에서 필요한 곳만 코드를 적용하여 동적으로 변경할 수 있다
        
        - 템플릿 엔진에는 JSP, Thymeleaf, Freemarker, Velocity등이 있다

## JSP로 회원 등록 페이지 만들기

- JSP로 회원 등록 폼 제공
    
    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
     <title>Title</title>
    </head>
    <body>
    	<form action="/jsp/members/save.jsp" method="post"> 
    		username: <input type="text" name="username" /> 
    		age: <input type="text" name="age" /> 
    		<button type="submit">전송</button>
    	</form>
    </body>
    </html>
    ```
    
    - JSP는 내부에서 서블릿으로 변환된다
    - <%@ ~ %> 이 부분은 JSP 문서라는 뜻이다. JSP 문서는 이렇게 시작해야한다
- JSP 회원 저장 페이지
    
    ```html
    <%@ page import="hello.servlet.domain.member.MemberRepository" %>
    <%@ page import="hello.servlet.domain.member.Member" %>
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <% 
    	// request, response 사용 가능
    	MemberRepository memberRepository = MemberRepository.getInstance();
    	System.out.println("save.jsp");
    	String username = request.getParameter("username");
    	int age = Integer.parseInt(request.getParameter("age"));
    	Member member = new Member(username, age);
    	System.out.println("member = " + member);
    	memberRepository.save(member);
    %>
    <html>
    <head>
        <meta charset="UTF-8">
    </head>
    <body>
    성공
      <ul>
          <li>id=<%=member.getId()%></li>
          <li>username=<%=member.getUsername()%></li>
          <li>age=<%=member.getAge()%></li>
    </ul>
    <a href="/index.html">메인</a>
    </body>
    </html>
    ```
    
    - <% ~ %> : 이 부분에 자바 코드를 입력한다
    - <%= ~ %> : 이 부분에 자바 코드를 출력한다

## 서블릿과 JSP의 한계

- 서블릿으로 개발할 때는 뷰 화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여 지저분하고 복잡했다
- JSP에서는 뷰를 생성하는 부분을 가져가고 중간중간 동적으로 변경이 필요한 부분에 자바 코드를 적용했다
- 이 경우 코드의 상위 절반은 비즈니스 로직, 나머지 하위 절반은 HTML 뷰 영역이다
- 자바 코드, 데이터 조회 리포지토리 등 다양한 코드가 JSP에 노출된다
    
    → JSP가 너무 많은 역할을 하고있다
    
    → 유지보수할 때 힘들다
    
    → **MVC 패턴의 등장**
    
    - 비즈니스 로직은 서블릿처럼 다른곳에서 처리하고, JSP는 목적에 맞게 화면을 그리는 일에만 집중

## MVC 패턴

- **필요성**
    - **너무 많은 역할**
        - 하나의 서블릿이나 JSP만으로 비즈니스 로직과 뷰 렌더링을 모두 처리하게 되면 너무 많은 역할을 하게되고 유지보수가 어려워진다
            - 비즈니스 로직 호출하는 부분에 변경이 발생해도 해당 코드를 손대야한다
    - **변경의 라이프 사이클**
        - 뷰 영역과 비즈니스 로직은 라이프 사이클이 다르다 (중요)
        - 예를 들어 UI를 일부 수정하는 일과 비즈니스 로직을 수정하는 일은 각각 다르게 발생할 가능성이 매우 높고 서로에게 영향을 주지 않는다
        - 이렇게 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수 하는데 좋지 않다
    - **기능 특화**
        - JSP와 같은 템플릿은 화면을 렌더링하는데 최적화 되어 있으므로 이 부분의 업무만 담당하는 것이 좋다
- 용어
    
    ![mvc 패턴](./%EC%84%9C%EB%B8%94%EB%A6%BF%2C%20JSP%2C%20MVC%20%ED%8C%A8%ED%84%B4/mvc.png)
    
    - **컨트롤러** : HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 담아 모델에 담는다
        - 컨트롤러에 비즈니스 로직을 둘 수 있지만 일반적으로 비즈니스 로직은 서비스 계층을 만들어 처리한다.
        - 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 역할을 담당한다
    - **모델 :** 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모델에 담아 전달해주어 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고 화면을 렌더링 하는 역할에만 집중할 수 있다.
    - **뷰** : 모델에 담겨있는 데이터를 사용하여 화면을 그리는 일에 집중한다.
- 서블릿을 컨트롤러로, JSP를 뷰로 사용
    
    ```java
    @Override
    protected void service(HttpServletRequest request, HttpServletResponseresponse)
    	throws ServletException, IOException {
    	  String viewPath = "/WEB-INF/views/new-form.jsp";
    	  RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
    ```
    
    - 이 경우 Model은 HttpServletRequest 객체를 사용한다.
    - 해당 객체 내부의 저장소를 사용하여 데이터를 보관하고 조회한다.
    - redirect vs foward
        - 리다이렉트: 실제 클라이언트에 응답이 나갔다가 클라이언트가 redirect 경로로 다시 요청
        - 포워드: 서버 내부에서 일어나는 호출로 클라이언트는 인지하지 못한다.
    - 뷰
    
    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    	<head>
        <meta charset="UTF-8">
        <title>Title</title>
      </head>
    <body>
    <!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] --> 
    <form action="save" method="post">
    	username: <input type="text" name="username" /> 
    	age: <input type="text" name="age" /> 
    	<button type="submit">전송</button>
    </form>
    </body>
    </html>
    ```
    
- 한계
    - 포워드 중복
        - 뷰로 이동하는 코드가 항상 중복 호출되어야 한다
    - 사용하지 않는 코드
        - 일부 코드를 사용할 때도 있고 사용하지 않을 때도 있다
        - 이러한 부분은 테스트 케이스 작성을 어렵게한다
    - 공통 처리가 어렵다
        - 기능이 복잡해질수록 공통으로 처리해야 하는 부분이 점점 더 많이 증가한다
    - 이러한 부분들은 프론트 컨트롤러 패턴을 도입하여 해결할 수 있다
        
        → 스프링 MVC의 핵심