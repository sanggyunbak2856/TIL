# Http 응답데이터

태그: 스프링 MVC

## HttpServletResponse

- HTTP 응답 메시지 생성
    - HTTP 응답코드 지정
    - 헤더 생성
    - 바디 생성
- 편의 기능 제공
    - Content-type, 쿠키, 리다이렉트

## HTTP 응답 데이터

- 단순 텍스트, HTML
    - content-type을 text/html로 지정해야한다.
- API JSON
    - content-type을 application/json으로 지정해야한다
    - Jackson 라이브러리를 사용하여 JSON 객체를 문자로 변경할 수 있다.