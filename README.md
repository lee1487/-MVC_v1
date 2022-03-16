# 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 by 인프런 김영한 

## 서블릿 

### Hello 서블릿 
```
  스프링 부트 환경에서 서블릿을 등록하고 사용해보자. 
  
  참고 
    - 서블릿은 톰캣 같은 웹 애플리케이션 서버를 직접 설치하고, 그 위에 서블릿 코드를 
	  클래스 파일로 빌ㄹ드해서 올린 다음, 톰캣 서버를 실행하면 된다. 하지만 이 과정은 
	  매우 번거롭다. 스프링 부트는 톰캣 서버를 내장하고 있으므로, 톰캣 서버 설치 없이 
	  편리하게 서블릿 코드를 실행할 수 있다. 

  스프링 부트 서블릿 환경 구성 
    @ServletComponentScan 
	  - 스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 @ServletComponentScan을
	    지원한다. 다음과 같이 추가하자. 
	
	hello.servlet.ServletApplication 
	  - @ServletComponentScan 추가 
	
	서블릿 등록하기 
	  - 처음으로 실제 동작하는 서블릿 코드를 등록해보자. 
	  
	hello.servlet.basic.HelloServlet 
	  - @WebServlet 서블릿 애노테이션 
	    - name: 서블릿 이름 
		- urlPatterns: URL 매핑 
		- HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 다음 메서드를 실행한다.
		  - protected void service(HttpServletRequest request,
		    HttpServletResponse response)
	  
	  - 웹 브라우저 실행  
	    - http://localhost:8080/hello?username=world
		- 결과: hello world 

  HTTP 요청 메시지 로그로 확인하기 
    - 다음 설정을 추가하자. 
	  - application.properties에 
	    logging.level.org.apache.coyote.http11=debug
	- 서버를 다시 시작하고, 요청해보면 서버가 받은 HTTP 요청 메세지를 
	  출력하는 것을 확인할 수 있다.
	
	참고 
	  - 운영서버에 이렇게 모든 요청 정보를 다 남기면 성능저하가 발생할 수 있다. 
	    개발단계에서만 적용하자. 

  서블릿 컨테이너 동작 방식 설명 
    - 내장 톰캣 서버 생성
	- HTTP요청, HTTP 응답 메시지 
	- 웹 애플리케이션 서버의 요청 응답 구조 
	참고 
	  - HTTP 응답에서 Content-Length는 웹 애플리케이션 서버가 자동으로 
	    생성해준다. 

  welcome 페이지 추가 
    - 지금부터 개발할 내용을 편리하게 참고할 수 있도록 welcome 페이지를 만들어두자. 
	- webapp 경로에 index.html을 두면 http://localhost:8080호출시 
	  index.html 페이지가 열린다.
```

### HttpServletRequest - 개요
```
  HttpServletRequest 역할 
    - HTTP 요청 메시지를 개발자가 직접 파싱해서 사용해도 되지만, 매우 불편할 것이다. 
	  서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 
	  HTTP 요청 메시지를 파싱한다. 그리고 그 결과를 HttpServletRequest 객체에 
	  담사어 제공한다. 
	- HttpServletRequest를 사용하면 다음과 같은 HTTP 요청 메시지를 편리하게 
	  조회할 수 있다. 

  HTTP 요청 메시지 
    POST /save HTTP/1.1
	Host: localhost:8080
	Content-Type: application/x-www-form-urlencoded
	
	username=kim&age=20


  - START LINE 
    - HTTP 메소드 
	- URL 
	- 쿼리 스트링 
	- 스키마, 프로토콜 
  - 헤더 
    - 헤더 조회 
  - 바디 
    - form 파라미터 형식 조회 
	- message body 데이터 직접 조회 

    - HttpServletRequest 객체는 추가로 여러가지 부가기능도 함께 제공한다. 
	  임시 저장소 기능 
	    - 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능 
		  - 저장: request.setAttribute(name,value)
		  - 조회: request.getAttribute(name)
	  
	  세션 관리 기능 
	    - request.getSession(create:true)

  중요
    - HttpServletRequest, HttpServletResponse를 사용할 때 
	  가장 중요한 점은 이 객체들이 HTTP 요청 메시지, HTTP 응답 메시지를 
	  편리하게 사용하도록 도와주는 객체라는 점이다. 따라서 이 기능에 대해서 
	  깊이있는 이해를 하려면 HTTP 스펙이 제공하는 요청, 응답 메시지 자체를 
	  이해해야 한다. 
```

### HttpServletRequest - 기본 사용법 
```
  HttpServletRequest가 제공하는 기본 기능들을 알아보자. 
  
  참고 
    - 로컬에서 테스트하면 IPv6 정보가 나오는데, IPv4 정보를 보고 싶으면 
	  다음 옵션을 VM options에 넣어주면 된다. 
	  -Djava.net.preferIPv4Stack=true 

  지금까지 HttpServletRequest를 통해서 HTTP 메시지의 start-line,
  header 정보 조회 방법을 이해했다. 이제 본격적으로 HTTP 요청 데이터를 어떻게 
  조회하는지 알아보자.
```

### HTTP 요청 데이터 - 개요 
```
  HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법을 알아보자. 
  
  주로 다음 3가지 방법을 사용한다. 
    - GET - 쿼리 파라미터 
	  - /url?username=hello&age=20
	  - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달 
	  - 예) 검색, 필터, 페이징등에서 많이 사용하는 방식 
	
	- POST - HTML Form 
	  - content-type: application/x-www-form-urlencoded
	  - 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
	  - 예) 회원 가입, 상품 주문, HTML Form 사용 
	- HTTP message body에 데이터를 직접 담아서 요청 
	  - HTTP API에서 주로 사용, JSON, XML, TEXT
	  - 데이터 형식은 주로 JSON 사용 
	  - POST, PUT, PATCH
```

### HTTP 요청 데이터 - GET 쿼리 파라미터 
```
  다음 데이터를 클라이언트에서 서버로 전송해보자. 
  
  전달 데이터 
    - username=hello
	- age=20 

  - 메시지 바디 없이, URL의 쿼리 파라미터를 사용해서 데이터를 전달하자. 
    - 예) 검색, 필터, 페이징등에서 많이 사용하는 방식 
  - 쿼리 파라미터는 URL에 다음과 같이 ?를 시작으로 보낼 수 있다. 추가 
    파라미터는 &로 구분하면 된다. 
    - http://localhost:8080/request-param?username=hello&age=20

  - 서버에서는 HttpServletRequest가 제공하는 다음 메서드를 통해 쿼리 파라미터를 편리하게 
    조회할 수 있다.

  복수 파라미터에서 단일 파라미터 조회 
    - username=hello&username=kim과 같이 파라미터 이름은 하나인데, 값이 
	  중복이면 어떻게 될까? request.getParameter()는 하나의 파라미터 이름에 
	  대해서 단 하나의 값만 있을 때 사용해야 한다. 지금처럼 중복일 때는 
	  request.getParameterValues()를 사용해야 한다. 
	  참고로 이렇게 중복일 때 request.getParameter()를 사용하면 
	  request.getParameterValues()의 첫 번째 값을 반환한다.
``` 

### HTTP 요청 데이터 - POST HTML Form 
```
  이번에는 HTML의 Form을 사용해서 클라이언트에서 서버로 데이터를 전송해보자. 
  주로 회원가입, 상품 주문 등에서 사용하는 방식이다. 
  
  특징 
    - content-type: application/x-www-form-urlencoded
	- 메시지 바디에 쿼리 파라미터 형식으로 데이터를 전달한다. 

  src/main/webapp/basic/hello-form.html 생성 
    주의 
	  - 웹 브라우저가 결과를 캐시하고 있어서, 과거에 작성했던 html 결과가 
	    보이는 경우도 있다. 이때는 웹 브라우저의 새로 고침을 직접 
		선택해주면 된다. 물론 서버를 재시작 하지 않아서 그럴 수도 있다. 

  POST의 HTML Form을 전송하면 웹 브라우저는 다음 형식으로 HTTP 메시지를 
  만든다.(웹 브라우저 개발자 모드 확인) 
    - 요청 URL: http://localhost:8080/request-param
	- content-type: application/x-www-form-urlencoded
	- message body: username=hello&age=20

  application/x-www-form-urlencoded 형식은 앞서 GET에서 살펴본 
  쿼리 파라미터 형식과 같다. 따라서 쿼리 파라미터 조회 메서드를 그대로 사용하면 된다. 
  클라이언트(웹 브라우저) 입장에서는 두 방식에 차이가 있지만, 서버 입장에서는 둘의
  형식이 동일하므로, request.getParameter()로 편리하게 구분없이 조회할 수 있다. 
  
  정리하면 request.getParameter()는 GET URL 쿼리 파라미터 형식도 지원하고, 
  POST HTML Form 형식도 지원한다. 
  
  참고 
    - content-type은 HTTP 메시지 바디의 데이터 형식을 지정한다. 
	- GET URL 쿼리 파라미터 형식으로 클라이언트에서 서버로 데이터를 전달할 때는 
	  HTTP 메시지 바디를 사용하지 않기 때문에 content-type이 없다. 
	- POST HTML Form 형식으로 데이터를 전달하면 HTTP 메시지 바디에 
	  해당 데이터를 포함해서 보내기 때문에 바디에 포함된 데이터가 어떤 형식인지 
	  content-type을 꼭 지정해야 한다. 이렇게 폼으로 데이터를 전송하는 
	  형식을 application/x-www-form-urlencoded라 한다.

  Postman을 사용한 테스트 
    - 이런 간단한 테스트에 HTML form을 만들기는 귀찮다. 이때는 Postman을 사용하면 된다. 

  Postman 테스트 주의사항
    - POST 전송시 
	  - Body -> x-www-form-urlencoded 선택 
	  - Headers에서 content-type:application/x-www-form-urlencoded로 
	    지정된 부분 꼭 확인
```

### HTTP 요청 데이터 - API 메시지 바디 - 단순 텍스트 
```
  HTTP message body에 데이터를 직접 담아서 요청 
    - HTTP API에서 주로 사용, JSON, XML, TEXT 
	- 데이터 형식은 주로 JSON 사용 
	- POST,PUT,PATCH
  먼저 가장 단순한 텍스트 메시지를 HTTP 메시지 바디에 담아서 전송하고, 읽어보자. 
  HTTP 메시지 바디의 데이터를 InputStream을 사용해서 직접 읽을 수 있다. 
  
  참고 
    - inputStream은 byte 코드를 반환한다. byte 코드를 우리가 읽을 수 
	  있는 문자(String)로 보려면 문자표(Charset)를 지정해주어야 한다. 
	  여기서는 UTF_8 Charset을 지정해주었다. 
```

### HTTP 요청 데이터 - API 메시지 바디 - JSON 
```
  이번에는 HTTP API에서 주로 사용하는 JSON 형식으로 데이터를 전달해보자. 
  
  JSON 형식 전송 
    - POST http://localhost:8080/request-body-json
	- content-type: application/json 
	- message body: {"username": "hello", "age": 20}
	- 결과: messageBody: {"username": "hello", "age": 20}

  참고 
    - JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 
	  Jackso, Gson 같은 JSON 변환 라이브러리를 추가해서 사용해야 한다. 
	  스프링 부트로 Spring MVC를 선택하면 기본으로 Jackson 라이브러리
	  (ObjectMapper)를 함께 제공한다. 
	- HTML form 데이터도 메시지 바디를 통해 전송되므로 직접 읽을 수 있다. 
	  하지만 편리한 파라미터 조회 기능(request.getParameter(..))
	  을 이미 제공하기 때문에 파라미터 조회 기능을 사용하면 된다. 
```

### HttpServletResponse - 기본 사용법 
```
  HttpServletResponse 역할 
    - HTTP 응답 메시지 생성 
	  - HTTP 응답코드 지정 
	  - 헤더 생성 
	  - 바디 생성
	- 편의 기능 제공 
	  - Content-Type, 쿠키, Redirect
```

### HTTP 응답 데이터 - 단순 텍스트, HTML
```
  HTTP 응답 메시지는 주로 다음 내용을 담아서 전달한다. 
    - 단순 텍스트 응답 
	  - 앞에서 살펴봄(writer.println("ok);)
	- HTML 응답 
	- HTTP API - MessageBody JSON 응답 

  HttpServletResponse - HTML 응답
    - HTTP 응답으로 HTML을 반환할 때는 content-type을 text/html로 
	  지정해야 한다. 
```

### HTTP 응답 데이터 - API JSON
```
  ResponseJsonServlet
    - HTTP 응답으로 JSON을 반환할 때는 content-type을 application/json로 
	  지정해야 한다. Jackson 라이브러리가 제공하는 objectMapper.writerValueAsString()을
	  사용하면 객체를 JSON 문자로 변경할 수 있다. 
	  
  참고 
    - application/json은 스펙상 utf-8 형식을 사용하도록 정의되어 있다. 그래서 스펙에서 
	  charset=utf-8과 같은 추가 파라미터를 지원하지 않는다. 따라서 application/json
	  이라고만 사용해야지 application/json;charset=utf-8이라고 전달하는 것은 
	  의미 없는 파라미터를 추가한 것이 된다. response.getWriter()를 사용하면 
	  추가 파라미터를 자동으로 추가해버린다. 이때는 response.getOutputStream()으로 
	  출력하면 그런 문제가 없다. 
```

## 서블릿, JSP, MVC 패턴 

### 회원 관리 웹 애플리케이션 요구사항
```
  회원 정보 
    이름: username 
	나이: age 

  기능 요구사항
    - 회원 저장 
	- 회원 목록 조회 

  회원 도메인 모델 
    - id는 Member를 회원 저장소에 저장하면 회원 저장소가 할당한다.
  회원 저장소 
    - 회원 저장소는 싱글톤 패턴을 적용했다. 스프링을 사용하면 스프링 빈으로 등록하면 되지만 
	  지금은 최대한 스프링 없이 순수 서블릿 만으로 구현하는 것이 목적이다. 
	- 싱글톤 패턴은 객체를 단 하나만 생성해서 공유해야 하므로 생성자를 private 접근자로 막아둔다.

  회원 저장소 테스트 코드 
    - 회원을 저장하고, 목록을 조회하는 테스트를 작성했다. 각 테스트가 끝날 때, 다음 테스트에 
	  영향을 주지 않도록 각 테스트의 저장소를 clearStore()를 호출해서 초기화했다.
```

### 서블릿으로 회원 관리 웹 애플리케이션 만들기
```
  이제 본격적으로 서블릿으로 회원 관리 웹 애플리케이션을 만들어보자. 
  가장 먼저 서블릿으로 회원 등록 HTML 폼을 제공해보자. 
  
  MemberFormServlet - 회원 등록 폼
    - MemberFormServlet은 단순하게 회원 정보를 입력할 수 있는 HTML Form을 
	  만들어서 응답한다. 자바 코드로 HTML을 제공해야 하므로 쉽지 않은 작업이다. 
	실행 
	  - HTML Form 데이터를 POST로 전송해도, 전달 받는 서블릿을 아직 만들지 
	    않았다. 그래서 오류가 발생하는 것이 정상이다. 

  이번에는 HTML Form에서 데이터를 입력하고 전송을 누르면 실제 회원 데이터가 
  저장되도록 해보자. 전송 방식은 POST HTML Form에서 학습한 내용과 같다. 
  
  MemberSaveServlet - 회원 저장
    - MemberSaveServlet은 다음 순서로 동작한다. 
	  1. 파라미터를 조회해서 Member 객체를 만든다.
	  2. Member 객체를 MemberRepository를 통해서 저장한다. 
	  3. Member 객체를 사용해서 결과 화면용 HTML을 동적으로 만들어서 응답한다. 
	실행 
	  - 데이터가 전송되고, 저장 결과를 확인할 수 있다. 

  이번에는 저장된 모든 회원 목록을 조회하는 기능을 만들어보자.
  
  MemberListServlet - 회원 목록  
    - MemberListServlet은 다음 순서로 동작한다. 
	  1. memberRepository.findAll()을 통해 모든 회원을 조회한다. 
	  2. 회원 목록 HTML을 for 루프를 통해서 회원 수 만큼 동적으로 생성하고 응답한다. 
	실행 
	  - 저장된 회원 목록을 확인할 수 있다. 

  템플릿 엔진으로 
    - 지금까지 서블릿과 자바 코드만으로 HTML을 만들어보았다. 서블릿 덕분에 동적으로 원하는 
	  HTML을 마음껏 만들 수 있다. 정적인 HTML 문서라면 화면이 계속 달라지는 회원의 
	  저장 결과라던가, 회원 목록 같은 동적인 HTML을 만드는 일은 불가능 할 것이다. 
	- 그런데, 코드에서 보듯이 이것은 매우 복잡하고 비효율 적이다. 자바 코드로 HTML을 
	  만들어 내는 것 보다 차라리 HTML 문서에서 동적으로 변경해야 하는 부분만 
	  자바 코드를 넣을 수 있다면 더 편리할 것이다. 이것이 바로 템플릿 엔진이 
	  나온 이유이다. 템플릿 엔진을 사용하면 HTML 문서에 필요한 곳만 코드를 적용해서 
	  동적으로 변경할 수 있다. 
	- 템플릿 엔진에는 JSP, Thymeleaf, Freemarker, Velocity등이 있다. 
	  다음 시간에는 JSP로 동일한 작업을 진행해보자. 

  참고 
    - JSP는 성능과 기능면에서 다른 템플릿 엔진과의 경쟁에서 밀리면서, 점점 사장되어 
	  가는 추세이다. 템플릿 엔진들은 가각 장단점이 있는데, 강의에서는 JSP는 앞부분에서 
	  잠깐 다루고, 스프링과 잘 통합되는 Thymeleaf를 사용한다. 

  Welcome 페이지 변경 
    - 지금부터 서블릿에서 JSP, MVC 패턴, 직접 만드는 MVC 프레임워크, 그리고 스프링까지 
	  긴 여정을 함께할 것이다. 편리하게 참고할 수 있도록 welcome 페이지를 변경하자
```

### JSP로 회원 관리 웹 애플리케이션 만들기
```
  JSP 라이브러리 추가 
  JSP를 사용하려면 먼자 다음 라이브러리를 추가해야 한다. 
    //JSP 추가 시작
	implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
	implementation 'javax.servlet:jstl'
	//JSP 추가 끝

  라이브러리를 추가하면 gradle refresh 해주자.
  
  회원 등록 폼 JSP
    - <%@ page contentType="text/html;charset=UTF-8" language="java" %>
	  - 첫 줄은 JSP문서라는 뜻이다. JSP 문서는 이렇게 시작해야 한다. 
	- 회원 등록 폼 JSP를 보면 첫 줄을 제외하고는 완전히 HTML와 똑같다. JSP는 서버 내부에서 
	  서블릿으로 변환되는데, 우리가 만들었던 MemberFormServlet과 거의 비슷한 모습으로 변환된다. 
	실행 
	  - 실행시 .jsp까지 함께 적어주어야 한다. 

  회원 저장 JSP
    - JSP는 자바 코드를 그대로 다 사용할 수 있다. 
	- <%@ page import="hello.servlet.domain.member.MemberRepository" %>
	  - 자바의 import문과 같다.
	- <% ~~ %>
	  - 이 부분에는 자바 코드를 입력할 수 있다. 
	- <%= ~~ %>
	  - 이 부분에는 자바 코드를 출력할 수 있다. 
	- 회원 저장 JSP를 보면, 회원 저장 서블릿 코드와 같다. 다른 점이 있다면, HTML을 중심으로 하고, 
	  자바 코드를 부분부분 입력해주었다. <% ~~ %>를 사용해서 HTML 중간에 자바 코드를 출력하고 있다.

  회원 목록 JSP 
    - 회원 리포지토리를 먼저 조회하고, 결과 List를 사용해서 중간에 <tr><td> HTML 태그를 반복해서 
	  출력하고 있다. 

  서블릿과 JSP의 한계 
    - 서블릿으로 개발할 때는 뷰(View)화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여서 
	  지저분하고 복잡했다.
	- JSP를 사용한 덕분에 뷰를 생성하는 HTML 작업을 깔끔하게 가져가고, 중간중간 동적으로 
	  변경이 필요한 부분에만 자바 코드를 적용했다. 그런데 이렇게 해도 해결되지 않는 
	  몇가지 고민이 남는다. 
	- 회원 저장 JSP를 보자. 코드의 상위 절반은 회원을 저장하기 위한 비즈니스 로직이고, 
	  나머지 하위 절반만 결과를 HTML로 보여주기 위한 뷰 영역이다. 화원 목록의 경우에도 
	  마찬가지다.
	- 코드를 잘 보면, JAVA 코드, 데이터를 조회하는 리포지토리 등등 다양한 코드가 모두 
	  JSP에 노출되어 있다. JSP가 너무 많은 역할을 한다. 이렇게 작은 프로젝트도 벌써 
	  머리가 아파오는데, 수백 수천줄이 넘어가는 JSP를 떠올려보면 정말 지옥과 
	  같을 것이다. (유지보수 지옥 썰) 

  MVC 패턴의 등장 
    - 비즈니스 로직은 서블릿 처럼 다른 곳에서 처리하고, JSP는 목적에 맞게 HTML로
	  화면(View)을 그리는 일에 집중하도록 하자ㅏ. 과거 개발자들도 모두 비슷한 
	  고민이 있었고, 그래서 MVC 패턴이 등장했다. 우리도 직접 MVC 패턴을 
	  적용해서 프로젝트를 리펙터링 해보자. 
```

### MVC 패턴 - 개요
```
  너무 많은 역할 
    - 하나의 서블릿이나 JSP만으로 비즈니스 로직과 뷰 렌더링까지 모두 처리하게 되면, 너무 많은 
	  역할을 하게되고, 결과적으로 유지보수가 어려워진다. 비즈니스 로직을 호출하는 부분에 변경이 
	  발생해도 해당 코드를 손대야 하고, UI를 변경할 일이 있어도 비즈니스 로직이 함께 있는 
	  해당 파일을 수정해야 한다. HTML 코드 하나 수정해야 하는데, 수백줄의 자바 코드가 
	  함께 있다고 상상해보라! 또는 비즈니스 로직을 하나 수정해야 하는데 수백 수천줄의 
	  HTML 코드가 함께 있다고 상상해보라.

  변경의 라이프 사이클 
    - 사실 이게 정말 중요한데, 진짜 문제는 둘 사이에 변경의 라이프 사이클이 다르다는 점이다. 
	  예를 들어서 UI를 일부 수정하는 일과 비즈니스 로직을 수정하는 일은 각각 다르게 발생할 
	  가능성이 매우 높고 대부분 서로에게 영향을 주지 않는다. 이렇게 변경의 라이프 사이클이 
	  다른 부분을 하나의 코드로 관리하는 것은 유지보수하기 좋지 않다. (물론 UI가 많이 
	  변하면 함께 변경될 가능성도 있다.) 

  기능 특화 
    - 특히 JSP 같은 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어 있기 때문에 이 부분의 
	  업무만 담당하는 것이 가장 효과적이다. 

  Model View Controller 
    - MVC 패턴은 지금까지 학습한 것 처럼 하나의 서블릿이나, JSP로 처리하던 것을 
	  컨트롤러(Controller)와 뷰(View)라는 영역으로 서로 역할을 나눈 것을 말한다. 
	  웹 애플리케이션은 보통 이 MVC 패턴을 사용한다. 
	컨트롤러 
	  - HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 
	    전달할 결과 데이터를 조회해서 모델에 담는다. 
	모델 
	  - 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 
	    덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링 하는 일에 
		집중할 수 있다. 
	뷰 
	  - 모델에 담겨있는 데이터를 사용해서 화면에 그리는 일에 집중한다. 여기서는 HTML을 
	    생성하는 부분을 말한다. 

  참고 
    - 컨트롤러에 비즈니스 로직을 둘 수도 있지만, 이렇게 되면 컨트롤러가 너무 많은 역할을 
	  담당한다. 그래서 일반적으로 비즈니스 로직은 서비스(Service)라는 계층을 별도로 
	  만들어서 처리한다. 그리고 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 역할을 
	  담당한다. 참고로 비즈니스 로직을 변경하면 비즈니스 로직을 호출하는 컨트롤러의 
	  코드도 변경될 수 있다. 앞에서는 이해를 돕기 위해 비즈니스 로직을 호출한다는 표현 
	  보다는, 비즈니스 로직이라 설명했다. 
```

### MVC 패턴 - 적용
```
  서블릿을 컨트롤러로 사용하고, JSP를 뷰로 사용해서 MVC 패턴을 적용해보자. 
  Model은 HttpServletRequest 객체를 사용한다. request는 내부에 데이터 저장소를 
  가지고 있는데, request.setAttribute(), request.getAttribute()를 
  사용하면 데이터를 보관하고, 조회할 수 있다. 
  
  회원 등록 
    - 회원 등록 폼 - 컨트롤러 
	  - hello.servlet.web.servletmvc.MvcMemberFormServlet
	    - dispatcher.forward()
		  - 다른 서블릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서 다시 
		    호출이 발생한다. 
	
	참고 
	  - /WEB-INF
	    - 이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다. 우리가 기대하는 것은 
		  항상 컨트롤러를 통해서 JSP를 호출하는 것이다. 
	
	  - redirect vs forward 
	    - 리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가, 클라이언트가 redirect 
		  경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다. 
		  반면에 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다. 

	- 회원 등록 폼 - 뷰
	  - main/webapp/WEB-INF/views/new-form.jsp
	    - 여기서 form의 action을 보면 절대 경로(로 시작)이 아니라 상대경로(로 시작X)하는 것을 
		  확인할 수 있다. 이렇게 상대 경로를 사용하면 폼 전송시 현재 URL이 속한 계층 경로 
		  + save가 호출된다. 
		- 현재 계층 경로: /servlet-mvc/members
		- 결과: /servlet-mvc/members/save
	주의 
	  - 이후 코드에서 해당 jsp를 계속 사용하기 때문에 상대경로를 사용한 부분을 그대로 유지해야 한다.

  회원 저장 
    - 회원 저장 - 컨트롤러 
	  - MvcMemberSaveServlet
	    - HttpServletRequest를 Model로 사용한다. 
		- request가 제공하는 setAttribute()를 사용하면 request 객체에 데이터를 보관해서 
		  뷰에 전달할 수 있다. 
		- 뷰는 request.getAttribute()를 사용해서 데이터를 꺼내면 된다. 
	- 회원 저장 - 뷰 
	  - main/webapp/WEB-INF/views/save-result.jsp
	    - <%= request.getAttribute("member")%>로 모델에 저장한 member 객체를 
		  꺼낼 수 있지만, 너무 복잡해진다. 
		- JSP는 ${}문법을 제공하는데, 이 분법을 사용하면 request의 attribute에 담긴 
		  데이터를 편리하게 조회할 수 있다. 
	
	- MVC 덕분에 컨트롤러 로직과 뷰 로직을 확실하게 분리한 것을 확인할 수 있다. 향후 화면에 수정이 
	  발생하면 뷰 로직만 변경하면 된다.

  회원 목록 조회 
    - 회원 목록 조회 - 컨트롤러 
	  - MvcMemberListServlet
	    - request 객체를 사용해서 List<Member> members를 모델에 보관했다.
	- 회원 목록 조회 - 뷰 
	  - main/webapp/WEB-INF/views/members.jsp
	    - 모델에 담아둔 members를 JSP가 제공하는 taglib 기능을 사용해서 반복하면서 출력했다. 
		  members 리스트에서 member를 순서대로 꺼내서 item 변수에 담고, 출력하는 과정을 반복한다. 
		- <c:forEach> 이 기능을 사용하려면 다음과 같이 선언해야 한다. 
		  - <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
		  - 해당 기능을 사용하지 않고 출력하면 매우 지저분한 것을 볼 수 있다.
		  - JSP와 같은 뷰 템플릿은 이렇게 화면을 렌더링 하는데 특화된 다양한 기능을 제공한다.
	참고 
	  - 앞서 설명했듯이 JSP를 학습하는 것이 이 강의의 주 목적이 아니다. JSP가 더 궁금한 분들은 
	    이미 수 많은 자료들이 있으므로 JSP로 검색하거나 관련된 책을 참고하길 바란다. (반나절이면 대부분의 
		기능을 학습할 수 있다.)
```

### MVC 패턴 - 한계
```
  MVC 패턴을 적용한 덕분에 컨트롤러의 역할과 뷰를 렌더링 하는 역할을 명확하게 구분할 수 있다. 
  특히 뷰는 화면을 그리믄 역할에 충실한 덕분에, 코드가 깔끔하고 직관적이다. 단순하게 모델에서 필요한 
  데이터를 꺼내고, 화면을 만들면 된다. 그런데 컨트롤러는 딱 봐도 중복이 많고, 필요하지 않는 코드들도 많아 보인다. 
  
  MVC 컨트롤러의 단점 
    - 포워드 중복 
	  - View로 이동하는 코드가 항상 중복 호출되어야 한다. 물론 이부분을 메서드로 공통화해도 되지만, 
	    해당 메서드도 항상 직접 호출해야 한다. 
	- ViewPath 중복 
	  - prefix: /WEB-INF/views
	  - suffix: .jsp
	  - 그리고 만약 jsp가 아닌 thymeleaf 같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야 한다. 
	- 사용하지 않는 코드 
	  - 다음 코드를 사용할 때도 있고, 사용하지 않을 때도 있다. 특히 response는 현재 코드에서 
	    사용되지 않는다. 
		- HttpServletRequest request, HttpServletResponse response
	  - 그리고 이런 HttpServletRequest, HttpServletResponse를 사용하는 코드는 
	    테스트 케이스를 작성하기도 어렵다. 
	- 공통 처리가 어렵다. 
	  - 기능이 복잡해질수록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다. 
	    단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 해당 메서드를 항상 호출해야 
		하고, 실수로 호출하지 않으면 문제가 될 것이다. 그리고 호출하는 것 자체도 중복이다. 
	
	- 정리하면 공통 처리가 어렵다는 문제가 있다. 
	  - 이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 수문장 
	    역할을 하는 기능이 필요하다.ㅏ 프론트 컨트롤러(Front Controller)패턴을 
		도입하면 이런 문제를 깔끔하게 해결할 수 있다.(입구를 하나로!)
	  - 스프링 MVC의 핵심도 바로 이 프론트 컨트롤러에 있다. 
```

## MVC 프레임워크 만들기 

### 프론트 컨트롤러 패턴 소개 
```
  FrontController 패턴 특징 
    - 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음 
	- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출 
	- 입구를 하나로!
	- 공통 처리 가능 
	- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨 

  스프링 웹 MVC와 프론트 컨트롤러 
    - 스프링 웹 MVC의 핵심도 바로 FrontController
	- 스프링 웹 MVC의 DispatcherServlet이 FrontController 패턴으로 구현되어 있음 
```

### 프론트 컨트롤러 도입 - v1
```
  프론트 컨트롤러를 단계적으로 도입해보자. 
  이번 목표는 기존 코드를 최대한 유지하면서, 프론트 컨트롤러를 도입하는 것이다.
  먼저 구조를 맞추어두고 점진적으로 리펙터링 해보자.
  
  ControllerV1
    - 서블릿과 비슷한 모양의 컨트롤러 인터페이스를 도입한다. 각 컨트롤러들은 이 인터페이스를 
	  구현하면 된다. 프론트 컨트롤러는 이 인터페이스를 호출해서 구현과 관계없이 로직의 
	  일관성을 가져갈 수 있다. 
	- 이제 이 인터페이스를 구현한 컨트롤러를 만들어보자. 지금 단계에서는 기존 로직을 최대한 
	  유지하는게 핵심이다.

  MemberFormControllerV1 - 회원 등록 컨트롤러
  MemberSaveControllerV1 - 회원 저장 컨트롤러
  MemberListControllerV1 - 회원 목록 컨트롤러
    - 내부 로직은 기존 서블릿과 거의 같다. 
	  이제 프론트 컨트롤러를 만들어보자.

  FrontControllerServletV1 - 프론트 컨트롤러
    - 프론트 컨트롤러 분석 
	  - urlPatterns
	    - urlPatterns = "/front-controller/v1/*"
		  - /front-controller/v1를 포함한 하위 모든 요청은 
		    이 서블릿에서 받아들인다.. 
		- 예) /front-controller/v1, /front-controller/v1/a,
		  /front-controller/v1/a/b
	
	  controllerMap
	    - key: 매핑 URL 
		- value: 호출될 컨트롤러 
	
	  service() 
	    - 먼저 requestURI를 조회해서 실제 호출할 컨트롤러를 controllerMap에서 찾는다. 
		  만약 없다면 404(SC_NOT_FOUND) 상태 코드를 반환한다. 
		- 컨트롤러를 찾고 controller.process(request,response);을 
		  호출해서 해당 컨트롤러를 실행한다. 
	
	  JSP 
	    - JSP는 이전 MVC에서 사용했던 것을 그대로 사용한다.
```

### View 분리 - v2 
```
  모든 컨트롤러에서 뷰로 이동하는 부분에 중복이 있고, 깔끔하지 않다.
    String viewPath = "/WEB-INF/views/new-form.jsp";
	RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
	dispatcher.forward(request, response);
  이 부분을 깔끔하게 분리하기 위해 별도로 뷰를 처리하는 객체를 만들자. 
  
  MyView
    - 뷰 객체는 이후 다른 버전에서도 함께 사용하므로 패키지 위치를 frontcontroller에 두었다. 
	- 코드만 봐서는 어떻게 활용하는지 아직 감이 안올 것이다. 다음 버전의 
	  컨트롤러 인터페이스를 만들어보자. 컨트롤러가 뷰를 반환하는 특징이 있다. 

  ControllerV2
  MemberFormControllerV2 - 회원 등록 폼
    - 이제 각 컨트롤러는 복잡한 dispatcher.forward()를 직접 생성해서 호출하지 
	  않아도 된다. 단순히 MyView 객체를 생성하고 거기에 뷰 이름만 넣고 반환하면 된다. 
	- ControllerV1을 구현한 클래스와 ControllerV2를 구현한 클래스를 비교해보면, 
	  이 부분의 중복이 확실하게 제거된 것을 확인할 수 있다.
  MemberSaveControllerV2 - 회원 저장
  MemberListControllerV2 - 회원 목록
  
  프론트 컨트롤러 V2
    - ControllerV2의 반환 타입이 MyView이므로 프론트 컨트롤러는 컨트롤러의 호출 결과로 
	  MyView를 반환받는다. 그리고 view.render()를 호출하면 forward 로직을 
	  수행해서 JSP가 실행된다. 
	- MyView.render()
	  - 프론트 컨트롤러의 도입으로 MyView 객체의 render()를 호출하는 부분을 모두 
	    일관되게 처리할 수 있다. 각각의 컨트롤러는 MyView 객체를 생성만 해서 반환하면 된다.
```

### Model 추가 - v3 
```
  서블릿 종속성 제거 
    - 컨트롤러 입장에서 HttpServletRequest, HttpServletResponse이 꼭 필요할까?
	  요청 파라미터 정보는 자바의 Map으로 대신 넘기도록 하면 지금 구조에서는 컨트롤러가 
	  서블릿 기술을 몰라도 동작할 수 있다. 
	- 그리고 request 객체를 Model로 사용하는 대신에 별도의 Model 객체를 만들어서 
	  반환하면 된다. 우리가 구현하는 컨트롤러가 서블릿 기술을 전혀 사용하지 않도록 변경해보자. 
	  이렇게 하면 구현 코드도 매우 단순해지고, 테스트 코드 작성이 쉽다. 

  뷰 이름 중복 제거 
    - 컨트롤러에서 지정하는 뷰 이름에 중복이 있는 것을 확인할 수 있다. 
	  컨트롤러는 뷰의 논리 이름을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 
	  단순화하자. 
	- 이렇게 해두면 향후 뷰의 폴더 위치가 함께 이동해도 프론트 컨트롤러만 고치면 된다. 
	  - /WEB-INF/views/new-form.jsp -> new-form
	  - /WEB-INF/views/save-result.jsp -> save-result
	  - /WEB-INF/views/members.jsp -> members

  ModelView
    - 지금까지 컨트롤러에서 서블릿에 종속적인 HttpServletRequest를 사용했다. 
	  그리고 Model도 request.setAttribute()를 통해 데이터를 저장하고 
	  뷰에 전달했다. 서블릿의 종속성을 제거하기 위해 Model을 직접 만들고, 추가로 
	  View 이름까지 전달하는 객체를 만들어보자. (이번 버전에서는 컨트롤러에서 
	  HttpServletRequest를 사용할 수 없다. 따라서 직접 
	  request.setAttribute()를 호출할 수도 없다. 따라서 Model이 
	  별도로 필요하다.)
	- 참고로 ModelView 객체는 다른 버전에서도 사용하므로 패키지를 frontcontroller에 둔다.
	- 뷰의 이름과 뷰를 렌더링할 때 필요한 model 객체를 가지고 있다. model은 
	  단순히 map으로 되어 있으므로 컨트롤러에서 뷰에 필요한 데이터를 key, value로 넣어주면 된다. 

  ControllerV3
    - 이 컨트롤러는 서블릿 기술을 전혀 사용하지 않는다. 따라서 구현이 매우 단순해지고, 
	  테스트 코드 작성시 테스트 하기 쉽다. 
	- HttpServletRequest가 제공하는 파라미터는 프론트 컨트롤러가 
	  paramMap에 담아서 호출해주면 된다. 응답 결과로 뷰 이름과 뷰에 전달할 
	  Model 데이터를 포함하는 ModelView 객체를 반환하면 된다. 

  MemberFormControllerV3 - 회원 등록 폼
    - ModelView를 생성할 때 new-form이라는 view의 논리적인 이름을 지정한다. 
	  실제 물리적인 이름은 프론트 컨트롤러에서 처리한다.

  MemberSaveControllerV3 - 회원 저장
    - paramMap.get("username")
	  - 파라미터 정보는 map에 담겨있다. map에서 필요한 요청 파라미터를 조회하면 된다. 
	- mv.getModel().put("member",member);
	  - 모델은 단순한 map이므로 모델에 뷰에서 필요한 member 객체를 담고 반환한다.

  MemberListControllerV3 - 회원 목록
  FrontControllerServletV3
    - view.render(mv.getModel(), request, response) 코드에서 컴파일 
	  오류가 발생할 것이다. 다음 코드를 참고해서 MyView 객체에 필요한 메서드를 추가하자. 
	
	- createParamMap()
	  - HttpServletRequest에서 파라미터 정보를 꺼내서 Map으로 변환한다. 
	    그리고 해당 Map(paramMap)을 컨트롤러에 전달하면서 호출한다. 
	- 뷰 리졸버 
	  - MyView view = viewResolver(viewName)
	    - 컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경한다. 
		  그리고 실제 물리 경로가 있는 MyView객체를 반환한다. 
		- 논리 뷰 이름: members
		- 물리 뷰 경로: /WEB-INF/views/member.jsp
	  view.render(mv.getModel(), request, response)
	    - 뷰 객체를 통해서 HTML 화면을 렌더링 한다. 
		- 뷰 객체의 render()는 모델 정보도 함께 받는다. 
		- JSP는 request.getAttribute()로 데이터를 조회하기 때문에, 
		  모델의 데이터를 꺼내서 request.setAttribute()로 담아둔다. 
		- JSP로 포워드 해서 JSP를 렌더링 한다. 
```

### 단순하고 실용적인 컨트롤러 - v4
```
  앞서 만든 v3 컨트롤러는 서블릿 종속성을 제거하고 뷰 경로의 중복을 제거하는 등, 
  잘 설계된 컨트롤러이다. 그런데 실제 컨트롤러 인터페이스를 구현하는 개발자 입장에서 보면, 
  항상 ModelView 객체를 생성하고 반환해야 하는 부분이 조금은 번거롭다. 
  좋은 프레임워크는 아키텍처도 중요하지만, 그와 더불어 실제 개발하는 개발자가 단순하고 
  편리하게 사용할 수 있어야 한다. 소위 실용성이 있어야 한다. 
  
  이번에는 v3를 조금 변경해서 실제 구현하는 개발자들이 매우 편리하게 개발할 수 있는 
  v4 버전을 개발해보자. 
  
  V4 구조 
    - 기본적인 구조는 V3와 같다. 대신에 컨트롤러가 ModelView를 반환하지 않고, 
	  viewName만 반환한다.

  ControllerV4
    - 이전 버전은 인터페이스에 ModelView가 없다. model 객체는 파라미터로
	  전달되기 때문에 그냥 사용하면 되고, 결과로 뷰의 이름만 반환해주면 된다. 
	- 실제 구현 코드를 보자 

  MemberFormControllerV4
    - 정말 단순하게 new-form이라는 뷰의 논리 이름만 반환하면 된다. 
  MemberSaveControllerV4
    - model.put("member",member) 모델이 파라미터로 전달되기 때문에 
	  모델을 직접 생성하지 않아도 된다. 
  MemberListControllerV4
  FrontControllerServletV4
    - FrontControllerServletV4는 사실 이전 버전과 거의 동일하다.
	- 모델 객체 전달 
	  - Map<String,Object> model = new HashMap<>(); //추가 
	  - 모델 객체를 프론트 컨트롤러에서 생성해서 넘겨준다. 컨트롤러에서 모델 객체에 
	    값을 담으면 여기에 그대로 담겨있게 된다. 
	- 뷰의 논리 이름을 직접 반환 
	  String viewName = controller.process(paramMap, model);
	  MyView view = viewResolver(viewName);
	  - 컨트롤러가 직접 뷰의 논리 이름을 반환하므로 이 값을 사용해서 실제 
	    물리 뷰를 찾을 수 있다. 

  정리 
    - 이번 버전의 컨트롤러는 매우 단순하고 실용적이다. 기존 구조에서 모델을 파라미터로 
	  넘기고, 뷰의 논리 이름을 반환한다는 작은 아이디어를 적용했을 뿐인데, 컨트롤러를 
	  구현하는 개발자 입장에서 보면 이제 군더더기 없는 코드를 작성할 수 있다. 
	- 또한 중요한 사실은 여기까지 한번에 온 것이 아니라는 점이다. 프레임워크가 점진적으로 
	  발전하는 과정 속에서 이런 방법도 찾을 수 있었다. 
	- 프레임워크나 공통 기능이 수고로워야 사용하는 개발자가 편리해진다.
```

### 유연한 컨트롤러1 - v5 
```
  만약 어떤 개발자는 ControllerV3 방식으로 개발하고 싶고, 어떤 개발자는 
  ControllerV4 방식으로 개발하고 싶다면 어떻게 해야할까? 
    public interface ControllerV3 {
		ModelView process(Map<String, String> paramMap);
	}
	
	public interface ControllerV4 {
		String process(Map<String, String> paramMap, 
		Map<String,Object> model);
	}

  어댑터 패턴 
    - 지금까지 우리가 개발한 프론트 컨트롤러는 한가지 방식의 컨트롤러 인터페이스만 
	  사용할 수 있다. ControllerV3, ControllerV4는 완전히 다른 
	  인터페이스이다. 따라서 호환이 불가능하다. 마치 v3는 110v이고, v4는 
	  220v 전기 콘센트 같은 것이다. 이럴 때 사용하는 것이 바로 어댑터이다. 
	  어댑터 패턴을 사용해서 프론트 컨트롤러가 다양한 방식의 컨트롤러를 
	  처리할 수 있도록 변경해보자. 

  V5 구조 
    - 핸들러 어댑터
	  - 중간에 어댑터 역할을 하는 어댑터가 추가되었는데 이름이 핸들러 어댑터이다. 
	    여기서 어댑터 역할을 해주는 덕분에 다양한 종류의 컨트롤러를 호출할 수 있다. 
	- 핸들러 
	  - 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경했다. 그 이유는 이제 어댑터가 
	    있기 때문에 꼭 컨트롤러의 개념 뿐만 아니라 어떠한 것이든 해당하는 종류의 
		어댑터만 있으면 다 처리할 수 있기 때문이다.

  MyHandlerAdapter
    - 어댑터는 이렇게 구현해야 한다는 어댑터용 인터페이스이다.
	- boolean supports(Object handler)
	  - handler는 컨트롤러를 말한다. 
	  - 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드다. 
	- ModelView handle(HttpServletRequest request, 
	  HttpServletResponse response, Object handler)
	  - 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다. 
	  - 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 ModelView를 
	    직접 생성해서라도 반환해햐 한다. 
	  - 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만 이제는 이 어댑터를 통해서 
	    실제 컨트롤러가 호출된다. 

  실제 어댑터를 구현해보자. 
  먼저 ControllerV3를 지원하는 어댑터를 구현하자. 
  
  ControllerV3HandlerAdapter 
    - 하나씩 분석해보자 
	public boolean supports(Object handler) {
		return (handler instanceof ControllerV3);
	}
	  - ControllerV3를 처리할 수 있는 어댑터를 뜻한다. 
    ControllerV3 controller = (ControllerV3) handler;
	Map<String, String> paramMap = createParamMap(request);
	ModelView mv = controller.process(paramMap);
	return mv;
	  - handler를 컨트롤러V3로 변환한 다음에 V3 형식에 맞도록 호출한다. 
	    supports()를 통해 ControllerV3만 지원하기 때문에 타입 변환은 
		걱정없이 실행해도 된다. ControllerV3는 ModelView를 반환하므로 
		그대로 ModelView를 반환하면 된다. 

  FrontControllerServletV5
    - 컨트롤러(Controller) -> 핸들러(Handler)
	  - 이전에는 컨트롤러를 직접 매핑해서 사용했다. 그런데 이제는 어댑터를 사용하기 
	    때문에, 컨트롤러 뿐만 아니라 어댑터가 지원하기만 하면, 어떤 것이라도 
		URL에 매핑해서 사용할 수 있다. 그래서 이름을 컨트롤러에서 더 넓은 
		범위의 핸들러로 변경했다. 
	- 생성자 
	  - 생성자는 핸들러 매핑과 어댑터를 초기화(등록)한다. 
	- 매핑 정보 
	  - private final Map<String, Object> handlerMappingMap 
	    = new HashMap<>();
	  - 매핑 정보의 값이 ControllerV3, ControllerV4 같은 인터페이스에서 
	    아무 값이나 받을 수 있는 Object로 변경되었다.
	- 핸들러 매핑 
	  - Object handler = getHandler(request)
	    - 핸들러 매핑 정보인 handlerMappingMap에서 URL에 매핑된 
		  핸들러(컨트롤러) 객체를 찾아서 반환한다. 
	- 핸들러를 처리할 수 있는 어댑터 조회
	  - MyHandlerAdapter adapter = getHandlerAdapter(handler)
	    - handler를 처리할 수 있는 어댑터를 adapter.supports(handler)
		  를 통해서 찾는다. handler가 ControllerV3 인터페이스를 구현했다면, 
		  ControllerV3HandlerAdapter 객체가 반환된다. 
	- 어댑터 호출 
	  - ModelView mv = adapter.handle(request,response,handler);
	    - 어댑터의 handle(request,response,handler) 메서드를 통해 
		  실제 어댑터가 호출된다. 어댑터는 handler(컨트롤러)를 호출하고 그 결과를 
		  어댑터에 맞추어 반환한다. 
		- ControllerV3HandlerAdapter의 경우 어댑터의 모양과 컨트롤러의 모양이 
		  유사해서 변환 로직이 단순하다. 

  정리 
    - 지금은 V3 컨트롤러를 사용할 수 있는 어댑터와 ControllerV3만 있어서 크게 감흥이 
	  없을 것이다. ControllerV4를 사용할 수 있도록 기능을 추가해보자.
```

### 유연한 컨트롤러2 - v5 
```
  FrontControllerV5에 ControllerV4 기능도 추가해보자. 
    - 핸들러 매핑(handlerMappingMap)에 ControllerV4를 사용하는 컨트롤러를 
	  추가하고, 해당 컨트롤러를 처리할 수 있는 어댑터인 ControllerV4HandlerAdpater도
	  추가하자. 

  ControllerV4HandlerAdpater
    - public boolean supports(Object handler) {
		return (handler instanceof ControllerV4);
	  }
	  - handler가 ControllerV4인 경우에만 처리하는 어댑터이다. 
	
	실행 로직 
	
	- ControllerV4 controller = (ControllerV4) handler;
	  Map<String, String> paramMap = createParamMap(request);
	  Map<String, Object> model = new HashMap<>();
	  String viewName = controller.process(paramMap, model);
	  - handler를 ControllerV4로 캐스팅 하고, paramMap, model을 
	    만들어서 해당 컨트롤러를 호출한다. 그리고 viewName을 반환 받는다. 
	
	어댑터 변환 
	
	ModelView mv = new ModelView(viewName);
	mv.setModel(model);
	return mv;
	  - 어댑터에서 이 부분이 단순하지만 중요한 부분이다. 
	  - 어댑터가 호출하는 ControllerV4는 뷰의 이름을 반환한다. 그런데 어댑터는 
	    뷰의 이름이 아니라 ModelView를 만들어서 반환해야 한다. 여기서 어댑터가 
		꼭 필요한 이유가 나온다. 
	  - ControllerV4는 뷰의 이름을 반환했지만, 어댑터는 이것을 ModelView로 
	    만들어서 형식을 맞추어 반환한다. 마치 110v 전기 콘센트를 220v 전기 
		콘센트로 변경하듯이! 

  어댑터와 ControllerV4
    public interface ControllerV4 {
		String process(Map<String, String> paramMap, Map<String, Object> model);
	}
	public interface MyHandlerAdapter {
		ModelView handle(HttpServletRequest request, HttpServletResponse response,
		Object handler) throws ServletException, IOException;
	}
```

### 정리 
```
  지금까지 v1 ~ v5로 점진적으로 프레임워크를 발전시켜 왔다. 
  지금까지 한 작업을 정리해보자. 
    - v1: 프론트 컨트롤러를 도입 
	  - 기존 구조를 퇴대한 유지하면서 프론트 컨트롤러를 도입 
	- v2: View 분류
	  - 단순 반복 되는 뷰 로직 분리 
	- v3: Model 추가 
	  - 서블릿 종속성 제거 
	  - 뷰 이름 중복 제거 
	- v4: 단순하고 실용적인 컨트롤러 
	  - v3와 거의 비슷 
	  - 구현 입장에서 ModelView를 직접 생성해서 반환하지 않도록 
	    편리한 인터페이스 제공 
	- v5: 유연한 컨트롤러 
	  - 어댑터 도입 
	  - 어댑터를 추가해서 프레임워크를 유연하고 확장성 있게 설계 

  여기에 애노테이션을 사용해서 컨트롤러를 더 편리하게 발전시킬 수도 있다. 만약 애노테이션을 
  사용해서 컨트롤러를 편리하게 사용할 수 있게 하려면 어떻게 해야할까? 바로 애노테이션을 
  지원하는 어댑터를 추가하면 된다. 다형성과 어댑터 덕분에 기존 구조를 유지하면서, 
  프레임워크의 기능을 확장할 수 있다. 
  
  스프링 MVC 
    - 여기서 더 발전시키면 좋겠지만, 스프링 MVC의 핵심 구조를 파악하는데 필요한 부분은 
	  모두 만들어보았다. 사실은 여러분이 지금까지 작성한 코드는 스프링 MVC 프레임워크의 
	  핵심 코드 축약 버전이고, 구조도 거의 같다. 
	- 스프링 MVC에는 지금까지 우리가 학습한 내용과 거의 같은 구조를 가지고 있다. 
```

## 스프링 MVC - 구조 이해 

### 스프링 MVC 전체 구조 
```
  직접 만든 MVC 프레임워크와 스프링 MVC를 비교해보자. 
    - 직접 만든 프레임워크 -> 스프링 MVC 비교 
	  - FrontController -> DispatcherServlet
	  - handlerMappingMap -> HandlerMapping
	  - MyHandlerAdapter -> HandlerAdapter
	  - ModelView -> ModelAndView
	  - viewResolver -> ViewResolver
	  - MyView -> View

  DispatcherServlet 구조 살펴보기 
    - org.springframework.web.servlet.DispatcherServlet
	- 스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다. 
	- 스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿(DispatcherServlet)이다.
	- 그리고 이 디스패처 서블릿이 바로 스프링 MVC의 핵심이다. 
	
	DispatcherServlet 서블릿 등록 
	  - DispatcherServlet도 부모 클래스에서 HttpServlet을 상속 받아서 
	    사용하고, 서블릿으로 동작한다. 
		- DispatcherServlet->FrameworkServlet->HttpServletBean
		  -> HttpServlet
	  - 스프링 부트는 DispatcherServlet을 서블릿으로 자동 등록하면서 모든 경로 
	    (urlPatterns="/")에 대해서 매핑한다. 
		- 참고: 더 자세한 경로가 우선순위가 높다. 그래서 기존에 등록한 서블릿도 함께 동작한다.
	
	요청 흐름 
	  - 서블릿이 호출되면 HttpServlet이 제공하는 service()가 호출된다. 
	  - 스프링 MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 
	    service()를 오버라이드해두었다. 
	  - FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 
	    DispatcherServlet.doDispatch()가 호출된다. 
	
  지금 부터 DispatcherServlet의 핵심 기능인 doDispatch() 코드를 
  분석해보자. 최대한 간단히 설명하기 위해 예외처리, 인터셉터 기능은 제외했다.
    - DispacherServlet.doDispatch()

  SpringMVC 구조 
    동작 순서 
	  1. 핸들러 조회 
	    - 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다. 
	  2. 핸들러 어댑터 조회 
	    - 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다. 
	  3. 핸들러 어댑터 실행 
	    - 핸들러 어댑터를 실행한다. 
	  4. 핸들러 실행 
	    - 핸들러 어댑터가 실제 핸들러를 실행한다. 
	  5. ModelAndView 반환
	    - 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 
		  반환한다. 
	  6. viewResolver 호출 
	    - 뷰 리졸버를 찾고 실행한다.
	  7. View 반환
	    - 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 
		  담당하는 뷰 객체를 반환한다. 
		  - JSP의 경우 InternalResourceView(JstlView)를 
		    반환하는데, 내부에 forward() 로직이 있다. 
	  8. 뷰 렌더링 
	    - 뷰를 통해서 뷰를 렌더링 한다. 
	
	인터페이스 살펴보기 
	  - 스프링 MVC의 큰 강점은 DispacherServlet 코드의 변경 없이, 
	    원하는 기능을 변경하거나 확장할 수 있다는 점이다. 지금까지 설명한 
		대부분을 확장 가능할 수 있게 인터페이스로 제공한다. 
	  - 이 인터페이스들만 구현해서 DispacherServlet에 등록하면 
	    여러분만의 컨트롤러를 만들 수도 있다. 
	
	주요 인터페이스 목록 
	  - 핸들러 매핑: org.springframework.web.servlet.HandlerMapping
	  - 핸들러 어댑터: org.springframework.web.servlet.HandlerAdapter
	  - 뷰 리졸버: org.springframework.web.servlet.ViewResolver
	  - 뷰: org.springframework.web.servlet.View

  정리 
    - 스프링 MVC는 코드 분량도 매우 많고, 복잡해서 내부 구조를 다 파악하는 것은 쉽지 않다.
	  사실 해당 기능을 직접 확장하거나 나만의 컨트롤러를 만드는 일은 없으므로 걱정하지 
	  않아도 된다. 왜냐하면 스프링 MVC는 전세계 수 많은 개발자들의 요구사항에 맞추어 
	  기능을 계속 확장해왔고, 그래서 여러분이 웹 애플리케이션을 만들 때 필요로 하는 
	  대부분의 기능이 이미 다 구현되어 있다. 
	- 그래도 이렇게 핵심 동작방식을 알아두어야 향후 문제가 발생했을 때 어떤 부분에서 
	  문제가 발생했는지 쉽게 파악하고, 문제를 해결할 수 있다. 그리고 확장 포인트가 
	  필요할 때, 어떤 부분을 확장해야 할지 감을 잡을 수 있다. 실제 다른 컴포넌트를 
	  제공하거나 기능을 확장하는 부분들은 강의를 진행하면서 조금씩 설명하겠다. 지금은 
	  전체적인 구조가 이렇게 되어 있구나 하고 이해하면 된다. 
	- 우리가 지금까지 함께 개발한 MVC 프레임워크와 유사한 구조여서 이해하기 
	  어렵지 않았을 것이다. 
```

### 핸들러 매핑과 핸들러 어댑터 
```
  핸들러 매핑과 핸들러 어댑터가 어떤 것들이 어떻게 사용되는지 알아보자. 
  지금은 전혀 사용하지 않지만, 과거에 주로 사용했던 스프링이 제공하는 간단한 컨트롤러 
  로 핸들러 매핑과 어댑터를 이해해보자. 
  
  Controller 인터페이스 
    - 과거 버전 스프링 컨트롤러 
	- org.springframework.web.servlet.mvc.Controller 
	public interface Controller {
		ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse 
		response) throws Exception;
	}
	- 스프링도 처음에는 이런 딱딱한 형식의 컨트롤러를 제공했다. 

  참고 
    - Controller 인터페이스는 @Controller 애노테이션과 전혀 다르다. 

  간단하게 구현해보자. 
  OldController 
    - @Component: 이 컨트롤러는 /springmvc/old-controller라는 
	  이름의 스프링 빈으로 등록되었다. 
	- 빈의 이름으로 URL을 매핑할 것이다. 
	실행 
	  - 콘솔에 OldController.handleRequest이 출력되면 성공이다. 
	  - 이 컨트롤러는 어떻게 호출될 수 있을까?
  
  스프링 MVC 구조 
    - 이 컨트롤러가 호출되려면 다음 2가지가 필요하다. 
	  - HandlerMapping(핸드러 매핑) 
	    - 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다. 
		- 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다. 
	  - HandlerAdapter(핸들러 어댑터) 
	    - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다. 
		- 예)Controller 인터페이스를 실행할 수 있는 핸들러 어댑터를 
		  찾고 실행해야 한다. 
	- 스프링은 이미 필요한 핸들러 매핑과 핸들러 어댑터를 대부분 구현해두었다. 개발자가 
	  직접 핸들러 매핑과 핸들러 어댑터를 만든는 일은 거의 없다. 

  스프링 부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터 
  (실제로는 더 많지만, 중요한 부분 위주로 설명하기 위해 일부 생략) 
  HandlerMapping
    - 0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 
	  @RequestMapping에서 사용 
	- 1 = BeanNameUrlHandlerMapping: 스프링 빈의 이름으로 핸들러를 찾는다. 
  HandlerAdapter 
	- 0 = RequestMappingHandlerAdapter: 애노테이션 기반의 컨트롤러인 
	  @RequestMapping에서 사용 
	- 1 = HttpRequestHandlerAdapter: HttpRequestHandler 처리 
	- 2 = SimpleControllerHandlerAdapter: Controller 인터페이스 
	  (애노테이션X, 과거에 사용)처리 
	
  핸들러 매핑도, 핸들러 어댑터도 모두 순서대로 찾고 만약 없으면 다음 순서로 넘어간다. 
  
  1. 핸들러 매핑으로 핸들러 조회 
    1-1. HandlerMapping을 순서대로 실행해서, 핸들러를 찾는다. 
	1-2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 
		 핸들러를 찾아주는 BeanNameUrlHandlerMapping가 실행에 성공하고 
		 핸들러인 OldController를 반환한다. 
  2. 핸들러 어댑터 조회 
    2-1. HandlerAdapter의 supports()를 순서대로 호출한다. 
	2-2. SimpleControllerHandlerAdapter가 Controller 인터페이스를 
		 지원하므로 대상이 된다. 
  3. 핸들러 어댑터 실행 
    3-1. 디스패치 서블릿이 조회한 SimpleControllerHandlerAdapter를 
		 실행하면서 핸들러 정보도 함께 넘겨준다. 
	3-2. SimpleControllerHandlerAdapter는 핸들러인 OldController를 
		 내부에서 실행하고, 그 결과를 반환한다. 

  정리 - OldController 핸들러 매핑, 어댑터 
    - OldController를 실행하면서 사용된 객체는 다음과 같다. 
	  - HandlerMapping = BeanNameUrlHandlerMapping
	  - HandlerAdapter = SimpleControllerHandlerAdapter

  HttpRequestHandler 
    - 핸들러 매핑과, 어댑터를 더 잘 이해하기 위해 Controller 인터페이스가 아닌 
	  다른 핸들러를 알아보자. HttpRequestHandler 핸들러(컨트롤러)는 
	  서블릿과 가장 유사한 형태의 핸들러이다. 
	
	HttpRequestHandler
	  - 간단하게 구현해보자. 
	MyHttpRequestHandler
	  실행 
	    - 웹 브라우저에 빈 화면이 나오고, 콘솔에 MyHttpRequestHandler.handlerRequest가 
		  출력되면 성공이다. 
	
	1. 핸들러 매핑으로 핸들러 조회 
	  1-1. HandlerMapping을 순서대로 실행해서, 핸들러를 찾는다. 
	  1-2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 
		   이름으로 핸들러를 찾아주는 BeanNameUrlHandlerMapping가 
		   실행에 성공하고 핸들러인 MyHttpRequestHandler를 반환한다.
	2. 핸들러 어댑터 조회 
	  2-1. HandlerAdapter의 supports()를 순서대로 호출한다. 
	  2-2. HttpRequestHandlerAdapter가 HttpRequestHandler
		   인터페이스를 지원하므로 대상이 된다. 
	3. 핸들러 어댑터 실행 
	  3-1. 디스패처 서블릿이 조회한 HttpRequestHandlerAdapter를 
		   실행하면서 핸들러 정보도 함께 넘겨준다. 
	  3-2. HttpRequestHandlerAdapter는 핸들러인 MyHttpRequestHandler를
		   내부에서 실행하고, 그 결과를 반환한다.
	정리 - MyHttpRequestHandler 핸들러매핑, 어댑터
	  - MyHttpRequestHandler를 실행하면서 사용된 객체는 다음과 같다. 
	    - HandlerMapping = BeanNameUrlHandlerMapping
		- HandlerAdapter = HttpRequestHandlerAdapter

  @RequestMapping
    - 조금 뒤에 설명하겠지만, 가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 
	  RequestMappingHandlerMapping, RequestMappingHandlerAdapter이다.
	- @RequestMapping의 앞글자를 따서 만든 이름인데, 이것이 바로 지금 스프링에서
	  주로 사용하는 애노테이션 기반의 컨트롤러를 지원하는 매핑과 어댑터이다. 실무에서는 
	  99.9% 이 방식의 컨트롤러를 사용한다.
```

### 뷰 리졸버 
```
  이번에는 뷰 리졸버에 대해서 자세히 알아보자. 
  OldController - View 조회할 수 있도록 변경
    - View를 사용할 수 있도록 다음 코드를 추가했다. 
	  - return new ModelAndView("new-form");
	실행 
	  - 웹 브라우저에 Whitelabel Error Page가 나오고, 콘솔에 
	    OldController.handleRequest이 출력될 것이다. 
	  - 실행해보면 컨트롤러는 정상 호출되지만, Whitelabel Error Page
	    오류가 발생한다. 
	application.properties에 다음 코드를 추가하자. 
	  spring.mvc.view.prefix=/WEB-INF/views
	  spring.mvc.view.suffix=.jsp

  뷰 리졸버 - InternalResourceViewResolver 
    - 스프링 부트는 InternalResourceViewResolver라는 뷰 리졸버를 
	  자동으로 등록하는데, 이때 application.properties에 등록한 
	  spring.mvc.view.prefix, spring.mvc.view.suffix 설정 
	  정보를 사용해서 등록한다. 
	- 참고로 권장하지는 않지만 설정 없이 다음과 같이 전체 경로를 주어도 동작하기는 한다. 
	  - return new ModelAndView("/WEB-INF/views/new-form.jsp");
	실행 
	  - 등록 폼이 정상 출력되는 것을 확인할 수 있다. 물론 저장 기능을 개발하지 
	    않았으므로 폼만 출력되고, 더 진행하면 오류가 발생한다.

  뷰 리졸버 동작 방식 
    스프링 MVC 구조 
	
	스프링 부트가 자동 등록하는 뷰 리졸버
	(실제로는 더 많지만, 중요한 부분 위주로 설명하기 위해 일부 생략) 
	  1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. 
	  (예: 엑셀 파일 생성 기능에 사용)
	  2 = InternalResourceViewResolver: JSP를 처리할 수 있는 
	  뷰를 반환한다. 
	
	1. 핸들러 어댑터 호출 
	  - 핸들러 어댑터를 통해 new-form이라는 논리 뷰 이름을 획득한다. 
	2. ViewResolver 호출 
	  - new-form 이라는 뷰 이름으로 viewResolver를 순서대로 호출한다. 
	  - BeanNameViewResolver는 new-form이라는 이름의 스프링 빈으로 
	    등록된 뷰를 찾아야 하는데 없다. 
	  - InternalResourceViewResolver가 호출된다. 
	3. InternalResourceViewResolver
	  - 이 뷰 리졸버는 InternalResourceView를 반환한다.
	4. 뷰 - InternalResourceView
	  - InternalResourceView는 JSP처럼 포워드 forward()를 호출해서 
	    처리할 수 있는 경우에 사용한다. 
	5. view.render()
	  - view.render()가 호출되고 InternalResourceView는 forward()를
	    사용해서 JSP를 실행한다. 

  참고 
    - InternalResourceViewResolver는 만약 JSTL 라이브러리가 있으면 
	  InternalResourceView를 상속받은 JstlView를 반환한다. JstlView는 
	  JSTL 태그 사용시 약간의 부가 기능이 추가된다. 
	- 다른 뷰는 실제 뷰를 렌더링하지만, JSP의 경우 forward()를 통해서 해당 JSP로 
	  이동(실행)해야 렌더링이된다. JSP를 제외한 나머지 뷰 템플릿들은 forward() 과정 
	  없이 바로 렌더링 된다.
	- Thymeleaf 뷰 템플릿을 사용해면 ThymeleafViewResolver를 등록해야 한다. 
	  최근에는 라이브러리만 추가하면 스프링 부트가 이런 작업도 모두 자동화해준다. 

  이제 본격적으로 스프링 MVC를 시작해보자.
```

### 스프링 MVC - 시작하기 
```
  스프링이 제공하는 컨트롤러는 애노테이션 기반으로 동작해서, 매우 유연하고 실용적이다. 과거에는 
  자바 언어에 애노테이션이 없기도 했고, 스프링도 처음부터 이런 유영한 컨트롤러를 제공한 것은 
  아니다. 
  
  @RequestMapping
    - 스프링 애노테이션을 활용한 매우 유연하고, 실용적인 컨트롤러를 만들었는데 이것이 바로 
	  @RequestMapping 애노테이션을 사용하는 컨트롤러이다. 다들 한번쯤 사용해보았을
	  것이다. 여담이지만 과거에는 스프링 프레임워크가 MVC 부분이 약해서 스프링을 사용
	  하더라도 MVC 웹 기술은 스트럿츠 같은 다른 프레임워크를 사용했었다. 그런데 
	  @RequestMapping기반의 애노테이션 컨트롤러가 등장하면서, MVC 부분도 
	  스프링의 완승으로 끝이 났다. 

  @RequestMapping
    - RequestMappingHandlerMapping
	- RequestMappingHandlerAdapter
	
	- 앞서 보았듯이 가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 
	  RequestMappingHandlerMapping, RequestMappingHandlerAdapter
	  이다. 
	- @RequestMapping의 앞글자를 따서 만든 이름인데, 이것이 바로 지금 스프링에서 주로 
	  사용하는 애노테이션 기반의 컨트롤러를 지원하는 핸들러 매핑과 어댑터이다. 실무에서는 
	  99.9% 이 방식의 컨트롤러를 사용한다. 
	
  그럼 이제 본격적으로 애노테이션 기반의 컨트롤러를 사용해보자 
  지금까지 만들었던 프레임워크에서 사용했던 컨트롤러를 @RequestMapping 기반의 
  스프링 MVC 컨트롤러 변경해보자.

  SpringMemberFormControllerV1 - 회원 등록 폼
    - @Controller 
	  - 스프링이 자동으로 스프링 빈으로 등록한다.(내부에 @Component 애노테이션이 있어서 
	    컴포넌트 스캔의 대상이 됨)
	  - 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다. 
	- @RequestMapping
	  - 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다. 애노테이션을 
	    기반으로 동작하기 때문에, 매서드의 이름은 임의로 지으면 된다. 
	- ModelAndView
	  - 모델과 뷰 정보를 담아서 반환하면 된다. 

  RequestMappingHandlerMapping은 스프링 빈 중에서 @RequestMapping 또는 
  @Controller가 클래스 레벨에 붙어 있는 경우에 매핑 정보로 인식한다. 
  따라서 다음 코드도 동일하게 동작한다.
	@Component //컴포넌트 스캔을 통해 스프링 빈으로 등록
	@RequestMapping
	public class SpringMemberFormControllerV1 {
	
		@RequestMapping("/springmvc/v1/members/new-form")
		public ModelAndView process() {
			return new ModelAndView("new-form");
		}
	}
  물론 컴포넌트 스캔 없이 스프링 빈으로 직접 등록해도 동작한다.
  
  SpringMemberSaveControllerV1 - 회원 저장
    - mv.addObject("member", member)
	  - 스프링이 제공하는 ModelAndView를 통해 Model 데이터를 추가할 때는 
	    addObject()를 사용하면 된다. 이 데이터는 이후 뷰를 렌더링할 때 사용된다.

  SpringMemberListControllerV1 - 회원 목록
```

### 스프링 MVC - 컨트롤러 통합
```
  @RequestMapping을 잘 보면 클래스 단위가 아니라 메서드 단위에 적용된 것을
  확인할 수 있다. 따라서 컨트롤러 클래스를 유연하게 하나로 통합할 수 있다.

  SpringMemberControllerV2  
    조합 
	  - 컨트롤러 클래스를 통합하는 것을 넘어서 조합도 가능하다.ㅏ 
	    다음 코드는 /springmvc/v2/members라는 부분에 중복이 있다. 
		- @RequestMapping("/springmvc/v2/members/new-form")
		- @RequestMapping("/springmvc/v2/members")
		- @RequestMapping("/springmvc/v2/members/save")
	
	  - 물론 이렇게 사용해도 되지만, 컨트롤러를 통합한 예제 코들르 보면 중복을 
	    어떻게 제거했는지 확인할 수 있다. 
	  - 클래스 레벨에 다음과 같이 @RequestMapping을 두면 메서드 레벨과 
	    조합이 된다. 
		@Controller
		@RequestMapping("/springmvc/v2/members")
		public class SpringMemberControllerV2 {}
	조합 결과 
	  - 클래스 레벨 @RequestMapping("/springmvc/v2/members")
	    - 메서드 레벨 @RequestMapping("/new-form")
		- 메서드 레벨 @RequestMapping("/save")
		- 메서드 레벨 @RequestMapping
```

### 스프링 MVC - 실용적인 방식 
```
  MVC 프레임워크 만들기에서 v3은 ModelAndView를 개발자가 직접 생성해서
  반환했기 때문에, 불편했던 기억이 날 것이다. 물론 v4를 만들면서 실용적으로 
  개선한 기억도 날 것이다. 
  
  스프링 MVC는 개발자가 편리하게 개발할 수 있도록 수 많은 편의 기능을 제공한다. 
  실무에서는 지금부터 설명하는 방식을 주로 사용한다. 
  SpringMemberControllerV3
    - Model 파라미터 
	  - save(), members()를 보면 Model을 파라미터로 받는 것을 
	    확인할 수 있다. 스프링 MVC도 이런 편의 기능을 제공한다. 
	- ViewName 직접 반환 
	  - 뷰의 논리 이름을 반환할 수 있다. 
	- @RequestParam 사용 
	  - 스프링은 HTTP 요청 파라미터를 @RequestParam으로 받을 수 있다. 
	    @RequestParam("username")은 request.getParameter("username")
		와 거의 같은 코드라 생각하면 된다. 물론 GET 쿼리 파라미터, 
		Post Form 방식을 모두 지원한다. 
	- @RequestMapping -> @GetMapping, @PostMapping
	  - @RequestMapping은 URL만 매칭하는 것이 아니라, 
	    HTTP Method도 함께 구분할 수 있다. 예를 들어서 URL이 
		/new-form이고, HTTP Method가 GET인 경우를 모두 만족하는 
		매핑을 하려면 다음과 같이 처리하면 된다. 
		@RequestMapping(value = "/new-form", method = RequestMethod.GET)
	  - 이것을 @GetMapping, @PostMapping으로 더 편리하게 
	    사용할 수 있다. 참고로 Get,Post, Put, Delete, Patch
		모두 애노테이션이 준비되어 있다. 
	  - @GetMapping 코드를 열어서 @RequestMapping 애노테이션을 
	    내부에 가지고 있는 모습을 확인하자.
```

## 스프링 MVC - 기본 기능 

### 로깅 간단히 알아보기 
```
  앞으로 로그를 사용할 것이기 때문에, 이번 시간에는 로그에 대해서 간단히 알아보자. 
  
  운영 시스템에서는 System.out.println()같은 시스템 콘솔을 사용해서 필요한 
  정보를 출력하지 않고, 별도의 로깅 라이브러리를 사용해서 로그를 출력한다. 
  참고로 로그 관련 라이브러리도 많고, 깊게 들어가면 끝이 없기 때문에, 여기서는 
  최소한의 사용 방법만 알아본다. 
  
  로깅 라이브러리 
    - 스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리 
	  (spring-boot-starter-logging)가 함께 포함된다.
	  스프링 부트 로깅 라이브러리는 기본으로 다음 로깅 라이브러리를 사용한다. 
	  - SLF4J
	  - Logback
  
  로그 라이브러리는 Logback, Log4J, Log4J2 등등 수 많은 라이브러리가 
  있는데, 그것을 통합해서 인터페이스로 제공하는 것이 바로 SLF4J 라이브러리다. 
  쉽게 이야기해서 SLF4J는 인터페이스이고, 그 구현체로 Logback 같은 로그 
  라이브러리를 선택하면 된다. 실무에서는 스프링 부트가 기본으로 제공하는 
  Logback을 대부분 사용한다. 
  
  로그 선언 
    - private Logger log = LoggerFactory.getLogger(getClass());
	- private static final Logger log = LoggerFactory.getLogger(Xxx.class)
	- @Slf4j: 롬복 사용 가능 

  로그 호출 
    - log.info("hello")
	- System.out.println("hello")
	- 시스템 콘솔로 직접 출력하는 것 보다 로그를 사용하면 다음과 같은 장점이 있다. 
	  실무에서는 항상 로그를 사용해야 한다.

  LogTestController
    - 매핑 정보 
	  - @RestController 
	    - @Controller는 반환 값이 String이면 뷰 이름으로 인식된다. 그래서 뷰를 
		  찾고 뷰가 랜더링 된다. @RestController는 반환 값으로 뷰를 찾는 것이 아니라 
		  HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다. 
		  @ResponseBody와 관련이 있는데, 뒤에서 더 자세히 설명한다. 
	- 테스트 
	  - 로그가 출력되는 포멧 확인 
	    - 시간, 로그 레벨, 프로세스 ID, 쓰레드 명, 클래스명, 로그 페이지 
	  - 로그 레벨 설정을 변경해서 출력 결과를 보자 
	    - LEVEL: TTRACE > DEBUG > INFO > WARN > ERROR
		- 개발 서버는 debug 출력 
		- 운영 서버는 info 추력 
	  - @Slf4j로 변경
	- 로그 레벨 설정
	  - application.properties
	    #전체 로그 레벨 설정(기본 info)
		logging.level.root=info
		#hello.springmvc 패키지와 그 하위 로그 레벨 설정
		logging.level.hello.springmvc=debug
	
	- 올바른 로그 사용법 
	  - log.debug("data=" + data)
	    - 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 
		  실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다 
	  - log.debug("data={}", data)
	    - 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 
		  의미없는 연산이 발생하지 않는다.

  로그 사용시 장점
    - 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다. 
	- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 
	  로그를 상황에 맞게 조절할 수 있다.ㅏ 
	- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 
	  남길 수 있다. 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다. 
	- 성능도 일반 System.out 보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 
	  실무에서는 꼭 로그를 사용해야 한다. 

  더 공부하실 분 
    - 로그에 대해서 더 자세한 내용은 slf4j, logback을 검색해보자. 
	  - SLF4J - http://www.slf4j.org
	  -	Logback - http://logback.qos.ch
	- 스프링 부트가 제공하는 로그 기능은 다음을 참고하자.
	  https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging
```

### 요청 매핑 
```
  MappingController
    매핑 정보(한번더) 
	  - @RestController 
	    - @Controller는 반환 값이 String이면 뷰 이름으로 인식된다. 그래서 뷰를 
		  찾고 뷰가 랜더링 된다. @RestController는 반환 값으로 뷰를 찾는 것이 아니라 
		  HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다. 
		  @ResponseBody와 관련이 있는데, 뒤에서 더 자세히 설명한다. 
	  - @RequestMapping("/hello-basic")
	    - /hello-basic URL 호출이 오면 이 메서드가 실행되도록 매핑한다. 
		- 대부분 속성을 배열[]로 제공하므로 다중 설정이 가능하다. {"/hello-basic", "/hello-go"}
	
	Postman으로 테스트 해보자 
	  - 둘다 허용 
	    - 다음 두가지 요청은 다른 URL이지만, 스프링은 다음 URL 요청들을 같은 요청으로 매핑한다.
		  - 매핑: /hello-basic
		  - URL 요청:/hello-basic, /hello-basic/
	  - HTTP 메서드 
	    - @RequestMapping에 method 속성으로 HTTP 메서드를 지정하지 않으면 
		  HTTP 메서드와 무관하게 호출된다. 
		- 모두 허용 GET,HEAD,POST,PUT,PATCH,DELETE
	  - HTTP 메서드 매핑 
	  
		@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
		public String mappingGetV1() {
			log.info("mappingGetV1");
			return "ok";
		}
		
		- 만약 여기에 POST 요청을 하면 스프링 MVC는 HTTP 405 상태코드
		  (Method Not Allowed)를 반환한다.
	
	  - HTTP 메서드 매핑 축약
	  
		@GetMapping(value = "/mapping-get-v2")
		public String mappingGetV2() {
			log.info("mapping-get-v2");
			return "ok";
		}
		
		- HTTP 메서드를 축약한 애노테이션을 사용하는 것이 더 직관적이다. 코드를 보면 내부에서 
		  @RequestMapping과 Method를 지정해서 사용하는 것을 확인할 수 있다. 
	
	  - PathVariable(경로 변수) 사용
	  
		@GetMapping("/mapping/{userId}")
		public String mappingPath(@PathVariable("userId") String data) {
			log.info("mappingPath userId={}", data);
			return "ok";
		}
		
		- 최근 HTTP API는 다음과 같이 리소스 경로에 식별자를 넣는 스타일을 선호한다. 
		  - /mapping/userA
		  - /users/1
		  - @RequestMapping은 URL 경로를 템플릿화 할 수 있는데, @PathVariable을 
		    사용하면 매칭 되는 부분을 편리하게 조회할 수 있다. 
		  - @PathVariable의 이름과 파라미터 이름이 같으면 생략할 수 있다.
	
	  - PathVariable 사용 - 다중
	  
		@GetMapping("/mapping/users/{userId}/orders/{orderId}")
		public String mappingPath(@PathVariable String userId, @PathVariable Long 
		orderId) {
			log.info("mappingPath userId={}, orderId={}", userId, orderId);
			return "ok";
		}
	  
	  - 특정 파라미터 조건 매핑 
	  
		@GetMapping(value = "/mapping-param", params = "mode=debug")
		public String mappingParam() {
			log.info("mappingParam");
			return "ok";
		}
		
		- 특정 파라미터가 있거나 없는 조건을 추가할 수 있다. 잘 사용하지 않는다.
	  
	  - 특정 헤더 조건 매핑 
	    
		@GetMapping(value = "/mapping-header", headers = "mode=debug")
		public String mappingHeader() {
			log.info("mappingHeader");
			return "ok";
		}
		
		- 파라미터 매핑과 비슷하지만, HTTP 헤더를 사용한다.
		  Postman으로 테스트 해야 한다. 
	
	  - 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume
	    
		@PostMapping(value = "/mapping-consume", consumes = "application/json")
		public String mappingConsumes() {
			log.info("mappingConsumes");
			return "ok";
		}
		
		- Postman으로 테스트 해야 한다. 
		  HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑한다.
		  만약 맞지 않으면 HTTP 415 상태코드(Unsupported Media Type)을 반환한다.
	
	  - 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce
	    @PostMapping(value = "/mapping-produce", produces = "text/html")
		public String mappingProduces() {
			log.info("mappingProduces");
			return "ok";
		}
		
		- HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다. 
		  마약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다.
```

### 요청 매핑 - API 예시 
```
  회원 관리를 HTTP API로 만든다 생각하고 매핑을 어떻게 하는지 알아보자 
  (실제 데이터가 넘어가는 부분은 생략하고 URL 매핑만)
  
  회원 관리 
    - 회원 목록 조회: GET 	/users
	- 회원 등록:	 POST	/users
	- 회원 조회:	 GET	/users/{userId}
	- 회원 수정:	 PATCH	/users/{userId}
	- 회원 삭제:	 DELETE /users/{userId}

  MappingClassController
    - /mapping은 강의의 다른 예제들과 구분하기 위해 사용했다.
	- @RequestMapping("/mapping/users")
	  - 클래스 레벨에 매핑 정보를 두면 메서드 레벨에서 해당 정보를 조합해서 사용한다.

  매핑 방법을 이해했으니, 이제부터 HTTP 요청이 보내는 데이터들을 스프링 MVC로 
  어떻게 조회하는지 알아보자.
```

### HTTP 요청 - 기본, 헤더 조회 
```
  애노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원한다.
  이번 시간에는 HTTP 헤더 정보를 조회하는 방법을 알아보자. 
  
  RequestHeaderController
    - HttpServletRequest
	- HttpServletResponse
	- HttpMethod: HTTP 메서드를 조회한다. org.springframework.http.HttpMethod
	- Locale: Locale 정보를 조회한다. 
	- @RequestHeader MultiValueMap<String,String> headerMap
	  - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다. 
	- @RequestHeader("host") String host
	  - 특정 HTTP 헤더를 조회한다. 
	  - 속성 
	    - 필수 값 여부: required
		- 기본 값 속성: defaultValue
	- @CookieValue(value="myCookie", required = false) String cookie
	  - 특정 쿠키를 조회한다.
	  - 속성 
	    - 필수 값 여부: required
		- 기본 값 속성: defaultValue
	

  MultiValueMap
    - MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다. 
	- HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
	  - keyA=value1&keyA=value2
	  MultiValueMap<String, String> map = new LinkedMultiValueMap();
	  
	  map.add("keyA", "value1");
	  map.add("keyA", "value2");
	  
	  //[value1,value2]
	  List<String> values = map.get("keyA");

  @Slf4j
    - 다음 코드를 자동으로 생성해서 로그를 선언해준다. 개발자는 편리하게 log라고 사용하면 된다.
	  private static final org.slf4j.Logger log = 
	  org.slf4j.LoggerFactory.getLogger(RequestHeaderController.class);
	  
```

### HTTP 요청 - 쿼리 파라미터, HTML Form 
```
  HTTP 요청 데이터 조회 - 개요 
  
  서블릿에서 학습했던 HTTP 요청 데이터를 조회 하는 방법을 다시 떠올려보자. 그리고 서블릿으로 학습했던 
  내용을 스프링이 얼마나 깔끔하고 효율적으로 바꾸어주는지 알아보자.
  
  HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법을 알아보자.
  
  클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 다음 3가지 방법을 사용한다. 
    - GET - 쿼리 파라미터 
	  - /url?username=hello&age=20
	  - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달 
	  - 예) 검색, 필터, 페이징등에서 많이 사용하는 방식 
	- POST - HTML Form
	  - content-type: application/x-www-form-urlencoded
	  - 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
	  - 예) 회원 가입, 상품 주문, HTML Form 사용 
	- HTTP message body에 데이터를 직접 담아서 요청 
	  - HTTP API에서 주로 사용, JSON, XML, TEXT
	  - 데이터 형식은 주로 JSON 사용 
	  - POST,PUT,PATCH
	
	하나씩 알아보자 

  요청 파라미터 - 쿼리 파라미터, HTML Form
    - HttpServletRequest의 request.getParameter()를 사용하면 
	  다음 두가지 요청 파라미터를 조회할 수 있다. 
	
	  - GET, 쿼리 파라미터 전송 
	    - http://localhost:8080/request-param?username=hello&age=20
	  - POST, HTML Form 전송 
	    - POST /request-param ...
		  content-type: application/x-www-form-urlencoded
		  
		  username=hello&age=20
	
	- GET 쿼리 파라미터 전송 방식이든, POST HTML Form 전송 방식이든 둘다 형식이 
	  같으므로 구분없이 조회할 수 있다. 이것을 간단히 요청 파라미터
	  (request parameter)조회라 한다ㅏ. 
	  
  지금부터 스프링으로 요청 파라미터를 조회하는 방법을 단계적으로 알아보자. 
  
  RequestParamController
    - request.getParameter()
	  - 여기서는 단순히 HttpServletRequest가 제공하는 방식으로 요청 파라미터를 조회했다. 
	- GET 실행 
	- POST Form 페이지 생성 
	  - 먼저 테스트용 HTML Form을 만들어야 한다.
	  - 리소스는 /resources/static 아래에 두면 스프링 부트가 자동으로 인식한다.
	- POST Form 실행 
	
	참고 
	  - Jar를 사용하면 webapp 경로를 사용할 수 없다. 이제부터 정적 리소스도 클래스 
	    경로에 함께 포함해야 한다.ㅏ 
```

### HTTP 요청 파라미터 - @RequestParam
```
  스프링이 제공하는 @RequestParam을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다. 
  
  requestParamV2
    - @RequestParam: 파라미터 이름으로 바인딩
	- @ResponseBody: View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력 
	- @RequestParam의 name(value)속성이 파라미터 이름으로 사용 
	- @RequestParam("username") String memberName
	  - request.getParameter("username")

  requestParamV3
    - HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능

  requestParamV4
    - String, int, Integer 등의 단순 타입이면 @RequestParam도 생략 가능 

  주의 
    - @RequestParam 애노테이션을 생략하면 스프링 MVC는 내부에서 required=false를 
	  적용한다. required 옵션은 바로 다음에 설명한다. 

  참고 
    - 이렇게 애노테이션을 완전히 생략해도 되는데, 너무 없는 것도 약간 과하다는 주관적
	  생각이 있다. @RequestParam이 있으면 명확하게 요청 파라미터에서 데이터를 
	  읽는 다는 것을 알 수 있다. 

  파라미터 필수 여부 - requestParamRequired
    - @RequestParam.required
	  - 파라미터 필수 여부 
	  - 기본값이 파라미터 필수(true)이다. 
	- /request-param 요청 
	  - username이 없으므로 400 예외가 발생한다. 
	주의! - 파라미터 이름만 사용 
	  - /request-param?username=
	    파라미터 이름만 있고 값이 없는 경우 -> 빈 문자로 통과 
	주의! - 기본형(primitive)에 null 입력 
	  - /request-param 요청 
	  - @RequestParam(required=false) int age
	  - null을 int에 입력하는 것은 불가능(500 예외 발생)
	    따라서 null을 받을 수 있는 Integer로 변경하거나, 또는 
		다음에 나오는 defaultValue 사용 

  기본 값 적용 - requestParamDefault
    - 파라미터에 값이 없는 경우 defaultValue를 사용하면 기본 값을 적용할 수 있다.
	  이미 기본 값이 있기 때문에 required는 의미가 없다. 
	 
	- defaultValue는 빈 문자의 경우에도 설정한 기본 값이 적용된다.
	  - /request-param?username=

  파라미터를 Map으로 조회하기 - requestParamMap
    - 파라미터를 Map, MultiValueMap으로 조회할 수 있다. 
	- @RequestParam Map
	  - Map(key=value)
	- @RequestParam MultiValueMap
	  - MultiValueMap(key=[value1,value2,...])
	    - ex) (key=userIds, value=[id1, id2])
	- 파라미터의 값이 1개가 확실하다면 Map을 사용해도 되지만, 그렇지 않다면 
	  MultiValueMap을 사용하자. 
```

### HTTP 요청 파라미터 - @ModellAttribute
```
  실제 개발을 하면 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주어야 한다. 
  보통 다음과 같이 코드를 작성할 것이다. 
    @RequestParam String username;
	@RequestParam int age;
	HelloData data = new HelloData();
	data.setUsername(username);
	data.setAge(age);
  
  스프링은 이 과정을 완전히 자동화해주는 @ModelAttribute 기능을 제공한다. 
  먼저 요청 파라미터를 바인딩 받을 객체를 만들자
  
  HelloData
    - 롬복 @Data
	  - @Getter, @Setter, @ToString, @EqualsAndHashCode,
	    @RequiredArgsConstructor를 자동으로 적용해 준다. 
		
  @ModelAttribute 적용 - modelAttributeV1
    - 마치 마법처럼 HelloData 객체가 생성되고, 요청 파라미터의 값도 모두 들어가 있다. 
	- 스프링 MVC는 @ModelAttribute가 있으면 다음을 실행한다. 
	  - HelloData 객체를 생성한다. 
	  - 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 
	    해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다. 
		- 예) 파라미터 이름이 username이면 setUsername() 메서드를 찾아서 
		  호출하면서 값을 입력한다. 
	
	프로퍼티 
	  - 객체에 getUsername(), setUsername() 메서드가 있으면, 이 객체는 
	    username이라는 프로퍼티를 가지고 있다. 
	  - username 프로퍼티의 값을 변경하면 setUsername()이 호출되고, 
	    조회하면 getUsername()이 호출된다. 
	
	바인딩 오류 
	  - age=abc 처럼 숫자가 들어가야 할 곳에 문자를 넣으면 BindException이
	    발생한다. 이런 바인딩 오류를 처리하는 방법은 검증 부분에서 다룬다.

  @ModelAttribute 생략 - modelAttributeV2
    - @ModelAttribute는 생략할 수 있다. 
	  그런데 @RequestParam도 생략할 수 있으니 혼란이 발생할 수 있다. 
	  
	- 스프링은 해당 생략시 다음과 같은 규칙을 적용한다. 
	  - String,int, Integer 같은 단순 타입 = @RequestParam
	  - 나머지 = @ModelAttribute(argument resolver로 지정해둔 타입 외)
	
	참고 
	  - argument resolver는 뒤에서 학습한다.
```

### HTTP 요청 메시지 - 단순 텍스트 
```
  서블릿에서 학습한 내용을 떠올려보자. 
    - HTTP message boyd에 데이터를 직접 담아서 요청 
	  - HTTP API에서 주로 사용 JSON, XML, TEXT
	  - 데이터 형식은 주로 JSON 사용 
	  - POST, PUT, PATCH

  요청 파라미터와 다르게, HTTP 메시지 바디를 통해 직접 데이터가 넘어오는 경우는 
  @RequestParam, @ModelAttribute를 사용할 수 없다.(물론 HTML Form
  형식으로 전달되는 경우는 요청 파라미터로 인정된다.)
    - 먼저 가장 단순한 텍스트 메시지를 HTTP 메시지 바디에 담아서 전송하고, 읽어보자.
	- HTTP 메시지 바디의 데이터를 InputStream을 사용해서 직접 읽을 수 있다. 

  RequestBodyStringController
    - Body -> row, Text 선택 

  Input, Output 스트림, Reader - requestBodyStringV2
    - 스프링 MVC는 다음 파라미터를 지원한다. 
	  - InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회 
	  - OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력 

  HttpEntity - requestBodyStringV3
    - 스프링 MVC는 다음 파라미터를 지원한다. 
	  - HttpEntity: HTTP header, body 정보를 편리하게 조회 
	    - 메시지 바디 정보를 직접 조회 
		- 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X,
		  @ModelAttribute X
	  - HttpEntity는 응답에도 사용 가능 
	    - 메시지 바디 정보 직접 반환 
		- 헤더 정보 포함 가능 
		- view 조회X
	
	- HttpEntity를 상속받은 다음 객체들도 같은 기능을 제공한다. 
	  - RequestEntity
	    - HttpMethod, url 정보가 추가, 요청에서 사용 
	  - ResponseEntity
	    - HTTP 상태 코드 설정 가능, 응답에서 사용 
		- return new ResponseEntity<String>("Hello World",
		  responseHeaders, HttpStatus,CREATED)
	
	참고 
	  - 스프링 MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 
	    전달해주는데, 이때 HTTP 메시지 컨버터(HttpMessageConverter)라는 
		기능을 사용한다. 이것은 조금 뒤에 HTTP 메시지 컨버터에서 자세히 설명한다. 

  @RequestBody - requestBodyStringV4
    - @RequestBody
	  - @RequestBody를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 
	    참고로 헤더 정보가 필요하다면 HttpEntity를 사용하거나 @RequestHeader를 
		사용하면 된다. 
	  - 이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 RequestParam,
	    @ModelAttribute와는 전혀 관계가 없다. 
	
  요청 파라미터 vs HTTP 메시지 바디 
    - 요청 파라미터를 조회하는 기능: @RequestParam, @ModelAttribute
	- HTTP 메시지 바디를 직접 조회하는 기능: @RequestBody

  @ResponseBody
    - @ResponseBody를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 
	  전달할 수 있다. 물론 이 경우에도 view를 사용하지 않는다.
```

### HTTP 요청 메시지 - JSON
```
  이번에는 HTTP API에서 주로 사용하는 JSON 데이터 형식을 조회해보자. 
  
  기존 서블릿에서 사용했던 방식과 비슷하게 시작해보자. 
  
  RequestBodyJsonController
    - HttpServletRequest를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 
	  문자로 변환한다.
	- 문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper를 사용해서 
	  자바 객체로 변환한다.

  requestBodyJsonV2 - @RequestBody 문자 변환
    - 이전에 학습했던 @RequestBody를 사용해서 HTTP 메시지에서 데이터를 꺼내고 
	  messageBody에 저장한다. 
	- 문자로 된 JSON 데이터인 messageBody를 objectMapper를 통해서  
	  자바 객체로 변환한다. 

  문자로 변환하고 다시 json으로 변환하는 과정이 불편하다. @ModelAttribute처럼 
  한번에 객체로 변환할 수는 없을까?
  
  requestBodyJsonV3 - @RequestBody 객체 변환
    - @RequestBody 객체 파라미터 
	  - @RequestBody HelloData data
	  - @RequestBody에 직접 만든 객체를 지정할 수 있다. 

  HttpEntity, @RequestBody를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 
  바디의 내용을 우리가 원하는 문자나 객체 등으로 반환해준다. 
  HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON 객체도 변환해주는데, 우리가 방금 V2
  에서 했던 작업을 대신 처리해준다. 
  자세한 내용은 뒤에서 HTTP 메시지 컨버터에서 다룬다. 
  
  @RequestBody는 생략 불가능 
    - @ModelAttribute에서 학습한 내용을 떠올려보자. 
    - 스프링은 @ModelAttribute, @RequestParam 해당 생략시 다음과 같은 
      규칙을 적용한다.
	  - String, int, Integer 같은 단순 타입 = @RequestParam
	  - 나머지 = @ModelAttribute(argument resolver로 지정해둔 타입 외)
	- 따라서 이 경우 HelloData에 @RequestBody를 생략하면 @ModelAttribute가
	  적용되어 버린다. HelloData -> @ModelAttribute HelloData data
	  따라서 생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다. 

  주의 
    - HTTP 요청시에 content-type이 application/json인지 꼭! 확인해야 한다.
	  그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다. 

  물론 앞서 배운 것과 같이 HttpEntity를 사용해도 된다. 
  requestBodyJsonV4 - HttpEntity
	@ResponseBody
	@PostMapping("/request-body-json-v4")
	public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
	 HelloData data = httpEntity.getBody();
	 log.info("username={}, age={}", data.getUsername(), data.getAge());
	 return "ok";
	}

  requestBodyJsonV5
    - @ResponseBody 
	  - 응답의 경우에도 @ResponseBody를 사용하면 해당 객체를 HTTP 메시지 바디에 
	    직접 넣어줄 수 있다. 물론 이경우에도 HttpEntity를 사용해도 된다. 
	- @RequestBody 요청 
	  - JSON 요청 -> HTTP 메시지 컨버터 -> 객체 
	- @ResponseBody 응답 
	  - 객체 -> HTTP 메시시 컨버터 -> JSON 응답 
```

### 응답 - 정적 리소스, 뷰 템플릿 
```
  응답 데이터는 이미 앞에서 일부 다룬 내용들이지만, 응답 부분에 초점을 맞추어서 정리해보자. 
  스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지이다. 
    - 정적 리소스 
	  - 예) 웹 브라우저에 정적인 HTML, css, js를 제공할 때는, 정적 리소스를 사용한다. 
	- 뷰 템플릿 사용 
	  - 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다. 
	- HTTP 메시지 사용 
	  - HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, 
	    HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다. 

  정적 리소스 
    - 스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다. 
	  - /static, /public, /resources, /META-INF/resources
	- src/main/resources는 리소스를 보관하는 곳이고, 또 클래스 패스의 
	  시작 경로이다. 따라서 다음 디렉토리에 리소스를 넣어두면 스프링 부트가 
	  정적 리소스로 서비스를 제공한다. 
	  - 정적 리소스 경로 
	    - src/main/resources/static
	  - 다음 경로에 파일이 들어있으면 
	    src/main/resources/static/basic/hello-form.html
		웹 브라우저에서 다음과 같이 실행하면 된다. 
		http://localhost:8080/basic/hello-form.html
	- 정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.

  뷰 템플릿 
    - 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다. 
	  일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것들도 가능하다. 
	  뷰 템플릿이 만들 수 있는 것이라면 뭐든지 가능하다. 
	  스프링 부트는 기본 뷰 템플릿 경로를 제공한다. 
	- 뷰 템플릿 경로 
	  - src/main/resources/templates
	- 뷰 템플릿 생성 
	  - src/main/resources/templates/response/hello.html

  ResponseViewController - 뷰 템플릿을 호출하는 컨트롤러
    - String을 반환하는 경우 - View or HTTP 메시지 
	  - @ResponseBody가 없으면 response/hello로 뷰 리졸버가 실행되어서 
	    뷰를 찾고, 렌더링 한다. @ResponseBody가 있으면 뷰 리졸버를 실행하지 않고 
		HTTP 메시지 바디에 직접 response/hello 라는 문자가 입력된다. 
	  - 여기서 뷰의 논리 이름인 response.hello를 반환하려면 다음 경로의 뷰 템플릿이 
	    렌더링 되는 것을 확인할 수 있다. 
	    - 실행: templates/response/hello.html
	- Void를 반환하는 경우 
	  - @Controller를 사용하고, HttpServletResponse, OutputStream(Writer)
	    같은 HTTP 메시지 바디를 처리하는 파라미터가 없으면 요청 URL을 
		참고해서 논리 뷰 이름으로 사용 
		- 요청 URL: /response/hello
		- 실행: templates/response/hello.html
	  - 참고로 이 방식은 명시성이 너무 떨어지고 이렇게 딱 맞는 경우도 많이 없어서, 권장하지 않는다. 
	- HTTP 메시지 
	  - @ResponseBody, HttpEntity를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, 
	    HTTP 메시지 바디에 직접 응답 데이터를 출력할 수 있다. 

  Thymeleaf 스프링 부트 설정 
    - 다음 라이브러리를 추가하면(이미 추가되어 있다.)
	  build.gradle
	    - `implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'`
	
	스프링 부트가 자동으로 ThymeleafViewResolver와 필요한 스프링 빈들을 등록한다. 
	그리고 다음 설정도 사용한다. 이 설정은 기본 값 이기 때문에 변경이 필요할 때만 설정하면 된다. 
	  application.properties
	    spring.thymeleaf.prefix=classpath:/templates
		spring.thymeleaf.suffix=.html

  참고 
    - 스프링 부트의 타임리프 관련 추가 설정은 다음 공식 사이트를 참고하자 (페이지 안에서 thymeleaf 검색)
	https://docs.spring.io/spring-boot/docs/2.4.3/reference/html/appendix-application-properties.html#common-application-properties-templating
```

### HTTP 응답 - HTTP API, 메시지 바디에 직접 입력 
```
  HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 
  같은 형식으로 데이터를 실어 보낸다. HTTP 요청에서 응답까지 대부분 다루었으므로 이번 시간에는 
  정리를 해보자 
  
  참고 
    - HTML이나 뷰 템플릿을 사용해도 HTTP 응답 메시지 바디에 HTML 데이터가 담겨서 전달된다. 
	  여기서 설명하는 내용은 정적 리소스나 뷰 템플릿을 거치지 않고, 직접 HTTP 응답 메시지를 
	  전달하는 경우를 말한다. 

  ResponseBodyController
    - responseBodyV1
	  - 서블릿을 직접 다룰 때 처럼 HttpServletResponse 객체를 통해서 
	    HTTP 메시지 바디에 직접 ok 응답 메시지를 전달한다. 
		response.getWriter().write("ok")
	- responseBodyV2
	  - ResponseEntity 엔티티는 HttpEntity를 상속 받았는데, 
	    HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다.
		ResponseEntity는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
	  - HttpStatus.CREATED로 변경하면 201응답이 나가는 것을 확인할 수 있다. 
	- responseBodyV3
	  - @ResponseBody를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 
	    통해서 HTTP 메시지를 직접 입력할 수 있다. ResponseEntity도 
		동일한 방식으로 동작한다. 
	- responseBodyJsonV1
	  - ResponseEntity를 반환한다. HTTP 메시지 컨버터를 통해서 
	    JSON 형식으로 변환되어서 반환된다. 
	- responseBodyJsonV2
	  - ResponseEntity는 HTTP 응답 코드를 설정할 수 있는데, 
	    @ResponseBody를 사용하면 이런 것을 설정하기 까다롭다. 
	  - @ResponseStatus(HttpStatus.OK) 애노테이션을 
	    사용하면 응답 코드도 설정할 수 있다. 
	  - 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 
	    프로그램 조건에 따라서 동적으로 변경하려면 ResponseEntity를 
		사용하면 된다. 

  @RestController
    - @Controller 대신에 @RestController 애노테이션을 사용하면, 
	  해당 컨트롤러에 모두 @ResponseBody가 적용되는 효과가 있다. 따라서 
	  뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다. 
	  이름 그대로 Rest API(HTTP API)를 만들 때 사용하는 컨트롤러이다. 
	  
	- 참고로 @ResponseBody는 클래스 레벨에 두면 전체에 메서드에 적용되는데, 
	  @RestController에노테이션 안에 @ResponseBody가 적용되어 있다. 
```

### HTTP 메시지 컨버터 
```
  뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 
  HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리하다. 
  
  HTTP 메시지 컨버터를 설명하기 전에 잠깐 과거로 돌아가서 스프링 입문 강의에서 설명했던 
  내용을 살펴보자. 
  
  @ResponseBody 사용원리 
    - @ResponseBody를 사용 
	  - HTTP의 BODY에 문자 내용을 직접 반환 
	  - viewResolver 대신에 HttpMessageConverter가 동작 
	  - 기본 문자처리: StringHttpMessageConverter
	  - 기본 객체처리: MappingJackson2HttpMessageConverter
	  - byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음
	
	참고 
	  - 응답의 경우 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 
	    정보 둘을 조합해서 HttpMessageConverter가 선택된다. 더 자세한 내용은 
		스프링 MVC 강의에서 설명하겠다. 

  이제 다시 돌아와서 
    - 스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다. 
	  - HTTP 요청: @RequestBody, HttpEntity(RequestEntity),
	  - HTTP 응답: @ResponseBody, HttpEntity(ResponseEntity),

  HTTP 메시지 컨버터 인터페이스 
    - org.springframework.http.converter.HttpMessageConverter
	- HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다. 
	  - canRead(), canWrite(): 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크 
	  - read(), write(): 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능 

  스프링 부트 기본 메시지 컨버터 (일부 생략)
    0 = ByteArrayHttpMessageConverter
	1 = StringHttpMessageConverter
	2 = MappingJackson2HttpMessageConverter

  스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 
  사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다. 
  
  몇가지 주요한 메시지 컨버터를 알아보자 
    - ByteArrayHttpMessageConverter: byte[] 데이터를 처리한다. 
	  - 클래스 타입: byte[], 미디어타입:*/*,
	  - 요청 예) @RequestBody byte[] data
	  - 응답 예) @ResponseBody return byte[] 
	    쓰기 미디어타입 application/octet-stream
	- StringHttpMessageConverter: String 문자로 데이터를 처리한다. 
	  - 클래스 타입: String, 미디어타입: */*,
	  - 요청 예) @RequestBody String data
	  - 응답 예) @ResponseBody return "ok" 
	    쓰기 미디어 타입 text/plain
	- MappingJackson2HttpMessageConverter: application/json
	  - 클래스 타입: 객체 또는 HashMap, 미디어타입 application/json 관련
	  - 요청 예) @RequestBody HelloData data
	  - 응답 예) @RequestBody return helloData 
	    쓰기 미디어타입 application/json 관련

  HTTP 요청 데이터 읽기 
    - HTTP 요청이 오고, 컨트롤러에서 @RequestBody, HttpEntity 파라미터를 사용한다. 
	- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead()를 호출한다. 
	  - 대상 클래스 타입을 지원하는가.
	    - 예) @RequestBody의 대상 클래스(byte[], String, HelloData)
	  - HTTP 요청의 Content-Type 미디어 타입을 지원하는가 
	    - 예) text/plain, application/json, */*
	- canRead() 조건을 만족하면 read()를 호출해서 객체를 생성하고, 반환한다. 

  HTTP 응답 데이터 생성 
    - 컨트롤러에서 @ResponseBody, HttpEntity로 값이 반환된다.
	- 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite()를 호출한다. 
	  - 대상 클래스 타입을 지원하는가.
	    - 예) return의 대상 클래스(byte[], String, HelloData)
	  - HTTP 요청의 Accept 미디어 타입을 지원하는가. ( 더 정확히는 
	    @RequestMapping의 produces)
		- 예) text/plain, application/json, */*
	- canWrite() 조건을 만족하면 write()를 호출해서 HTTP 응답 메시지 바디에
	  데이터를 생성한다. 
```

### 요청 매핑 헨들러 어뎁터 구조
```
  그렇다면 HTTP 메시지 컨버터는 스프링 MVC 어디쯤에서 사용되는 것일까?
  다음 그림에서는 보이지 않는다. 
  
  SpringMVC 구조 
    - 모든 비밀은 애노테이션 기반의 컨트롤러, 그러니까 @RequestMapping을 처리하는 
	  핸들러 어댑터인 RequestMappingHandlerAdapter(요청 매핑 핸들러 어댑터)에 있다. 

  RequestMappingHandlerAdapter 동작 방식 
  
    ArgumentResolver
	  - 생각해보면, 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다. 
	    HttpServletRequest, Model은 물론이고 @RequestParam,
		@ModelAttribute 같은 애노테이션 그리고 @RequestBody, HttpEntity
		같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 보여주었다. 
		이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 ArgumentResolver
		덕분이다. 
	  - 애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter는 
	    바로 이 ArgumentResolver를 호출해서 컨트롤러(핸들러)가 필요로 하는 
		다양한 파라미터의 값(객체)을 생성한다. 그리고 이렇게 파라미터의 값이 모두 
		준비되면 컨트롤러를 호출하면서 값을 넘겨준다.
	  - 스프링은 30개가 넘는 ArgumentResolver를 기본으로 제공한다. 
	    어떤 종류들이 있는지 살짝 코드로 확인만 해보자. 
	
	정확히는 HandlerMethodArgumentResolver인데 줄여서 ArgumentResolver라고 부른다. 
	  동작 방식 
	    - ArgumentResolver의 supportsParameter()를 호출해서 해당 파라미터를 
		  지원하는지 체크하고, 지원하면 resolveArgument()를 호출해서 실제 객체를 
		  생성한다. 그리고 이렇게 생성된 객체가 컨트롤러 호출시 넘어가는 것이다.
		- 그리고 원한다면 여러분이 직접 이 인터페이스를 확장해서 원하는 ArgumentResolver를 
		  만들 수도 있다. 실제 확장하는 예제는 향후 로그인 처리에서 진행하겠다. 
	
	ReturnValueHandler
	  - HandlerMethodReturnValueHandler를 줄여서 ReturnValueHandler라
	    부른다. ArgumentResolver와 비슷한데, 이것은 응답 값을 변환하고 처리한다. 
	  - 컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 
	    ReturnValueHandler 덕분이다. 어떤 종류들이 있는지 살짝 코드로
		확인만 해보자. 
	  - 스프링은 10여개가 넘는 ReturnValueHandler를 지원한다.
	    - 예) ModelAndView, @ResponseBody, HttpEntity, String

  HTTP 메시지 컨버터 
    HTTP 메시지 컨버터 위치 
	  - HTTP 메시지 컨버터는 어디쯤 있을까?
	    HTTP 메시지 컨버터를 사용하는 @RequestBody도 컨트롤러가 필요로 하는 
		파라미터의 값에 사용된다. @ResponseBody의 경우도 컨트롤러의 반환 값을
		이용한다.
	  - 요청의 경우 @RequestBody를 처리하는 ArgumentResolver가 있고, 
	    HttpEntity를 처리하는 ArgumentResolver가 있다. 
		이 ArgumentResolver들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 
		생성하는 것이다.(어떤 종류가 있는지 코드로 살짝 확인해보자)
	  - 응답의 경우 @ResponseBody와 HttpEntity를 처리하는 
	    ReturnValueHandler가 있다. 그리고 여기에서 HTTP 메시지 컨버터를 
		호출해서 응답 결과를 만든다. 
	  - 스프링 MVC는 @RequestBody, @ResponseBody가 있으면 
	    RequestResponseBodyMethodProcessor(ArgumentResolver)
		HttpEntity가 있으면 HttpEntityMethodProcessor(ArgumentResolver)
		를 사용한다. 
	  
	  참고 
	    - HttpMessageConverter를 구현한 클래스를 한번 확인해보자.
	
	확장 
	  - 스프링은 다음을 모두 인터페이스로 제공한다. 따라서 필요하면 언제든지 기능을 
	    확장할 수 있다. 
		- HandlerMethodArgumentResolver
		- HandlerMethodReturnValueHandler
		- HttpMessageConverter
	  - 스프링이 필요한 대부분의 기능을 제공하기 때문에 실제 기능을 확장할 일이 많지는 않다. 
	    기능 확장은 WebMvcConfigurer를 상속 받아서 스프링 빈으로 등록하면 된다. 
		실제 자주 사용하지는 않으니 실제 기능 확장이 필요할 때 WebMvcConfigurer를 
		검색해보자.
	    
```

## 스프링 MVC - 웹페이지 만들기 

### 요구사항 분석
```
  상품을 관리할 수 있는 서비스를 만들어보자. 
  
  상품 도메인 모델 
    - 상품 ID 
	- 상품명
	- 가격
	- 수량 

  상품 관리 기능 
    - 상품 목록
	- 상품 상세 
	- 상품 등록
	- 상품 수정 

  서비스 제공 흐름 
    요구사항이 정리되고 디자이너, 웹 퍼블리셔, 백엔드 개발자가 업무를 나누어 진행한다. 
	  - 디자이너
	    - 요구사항에 맞도록 디자인하고, 디자인 결과물을 웹 퍼블리셔에게 넘겨준다. 
	  - 웹 퍼블리셔
	    - 디자이너에게서 받은 디자인을 기반으로 HTML, CSS를 만들어
	      개발자에게 제공한다.
	  - 백엔드 개발자
	    - 디자이너, 웹 퍼블리셔를 통해서 HTML 화면이 나오기 전까지 시스템을 설계하고,
		  핵심 비즈니스 모델을 개발한다. 이후 HTML이 나오면 이 HTML을 뷰 템플릿으로 
		  변환해서 동적으로 화면을 그리고, 또 웹 화면의 흐름을 제어한다.
	
	참고 
	  - React,Vue.js 같은 웹 클라이언트 기술을 사용하고, 웹 프론트엔드 개발자가 
	    별도로 있으면, 웹 프론트엔드 개발자가 웹 퍼블리셔 역할까지 포함해서 하는 
		경우도 있다. 웹 클라이언트 기술을 사용하면, 웹 프론트엔드 개발자가 HTML을 
		동적으로 만드는 역할과 웹 화면의 흐름을 담당한다. 이 경우 백엔드 개발자는 
		HTML 뷰 템플릿을 직접 만지는 대신에, HTTP API를 통해 웹 클라이언트가 
		필요로 하는 데이터와 기능을 제공하면 된다. 
```

### 상품 도메인 개발 
```
  Item - 상품 객체 
  ItemRepository - 상품 저장소
  ItemRepositoryTest - 상품 저장소 테스트
```

### 상품 서비스 HTML 
```
  핵심 비즈니스 로직을 개발하는 동안, 웹 퍼블리셔는 HTML 마크업을 완료했다. 
  다음 파일들을 경로에 넣고 잘 동작하는지 확인해보자. 
  
  부트스트랩 
    - 참고로 HTML을 편리하게 개발하기 위해 부트스트랩을 사용했다. 
	  먼저 필요한 부트스트랩 파일을 설치하자. 
	- 부트스트랩 공식 사이트: https://getbootstrap.com
	- 부트스트랩을 다운로드 받고 압축을 풀자 
	  - 이동: https://getbootstrap.com/docs/5.0/getting-started/download/
	  - Compiled CSS and JS 항목을 다운로드 하자 
	  - 압축을 풀고 bootstrap.min.css를 복사해서 다음 폴더에 추가하자. 
	  - resources/static/css/bootstrap.min.css
	참고 
	  - 부트스트랩(Bootstrap)은 웹사이트를 쉽게 만들 수 있게 도와주는 HTML,CSS,JS 
	    프레임워크이다. 하나의 CSS로 휴대폰, 태블릿, 데스크탑까지 다양한 기기에서 작동한다. 
		다양한 기능을 제공하여 사용자가 쉽게 웹사이트를 제작, 유지, 보수할 수 있도록 도와준다.

  HTML, css 파일
	- /resources/static/css/bootstrap.min.css -> 부트스트랩 다운로드
	- /resources/static/html/items.html -> 아래 참조
	- /resources/static/html/item.html
	- /resources/static/html/addForm.html
	- /resources/static/html/editForm.html
	- 참고로 /resources/static에 넣어두었기 때문에 스프링 부트가 정적 리소스를 제공한다.
	  - http://localhost:8080/html/items.html
	  - 그런데 정적 리소스여서 해당 파일을 탐색기를 통해 직접 열어도 동작하는 것을 
	    확인할 수 있다. 
	
	참고 
	  - 이렇게 정적 리소스가 공개되는 /resources/static 폴더에 HTML을 넣어두면,
	    실제 서비스에서도 공개된다. 서비스를 운영한다면 지금처럼 공개할 필요없는 HTML을 
		두는 것은 주의하자.
		
	상품 목록 HTML
	  - resource/static/html/items.html
	상품 상세 HTML
	  - resources/static/html/item.html
	상품 등록 폼 HTML
	  - resources/static/html/addForm.html
	상품 수정 폼 HTML
	  - resources/static/html/editForm.html
```

### 상품 목록 - 타임리프
```
  본격적으로 컨트롤러와 뷰 템플릿을 개발해보자. 
  
  BasicItemController
    - 컨트롤러 로직은 itemRepository에서 모든 상품을 조회한 다음에 모델에 담는다. 
	  그리고 뷰 템플릿을 호출한다. 
	- @RequiredArgsConstructor
	  - final이 붙은 멤버 변수만 사용해서 생성자를 자동으로 만들어준다.
	  
	  public BasicItemController(ItemRepository itemRepository) {
		 this.itemRepository = itemRepository;
	  }
	  - 이렇게 생성자가 딱 1개만 있으면 스프링이 해당 생성자에 @Autowired로 
	    의존관계를 주입해준다. 
	  - 따라서 final 키워드를 빼면 안된다!, 그러면 ItemRepository 
	    의존관계 주입이 안된다.
	  - 스프링 핵심원리 - 기본편 강의 참고 
	
	테스트용 데이터 추가 
	  - 테스트용 데이터가 없으면 회원 목록 기능이 정상 동작하는지 확인하기 어렵다. 
	  - @PostConstruct: 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출된다. 
	  - 여기서는 간단히 테스트용 데이터를 넣기 위해서 사용했다. 
	
	items.html 정적 HTML을 뷰 템플릿(templates) 영역으로 복사하고 수정하자.
	  - /resource/static/items.html -> 복사 
	    -> /resources/templates/basic/items.html

  타임리프 간단히 알아보기 
    - 타임리프 사용 선언 
	  - <html xmlns:th="http://www.thymeleaf.org">
	- 속성 변경 - th:href 
	  - th:href="@{/css/bootstrap.min.css}"
	  - href="value1"을 th:href="value2"의 값으로 변경한다. 
	  - 타임리프 뷰 템플릿을 거치게 되면 원래 값을 th:xxx 값으로 변경한다. 
	    만약 값이 없다면 새로 생성한다.
	  - HTML을 그대로 볼 때는 href 속성이 사용되고, 뷰 템플릿을 거치면 th:href의 값이 
	    href로 대체되면서 동적으로 변경할 수 있다.
	  - 대부분의 HTML 속성을 th:xxx로 변경할 수 있다. 
	
	- 타임리프 핵심 
	  - 핵심은 th:xxx가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다.
	    th:xxx이 없으면 기존 html의 xxx 속성이 그대로 사용된다. 
	  - HTML을 파일로 직접 열었을 때, th:xxx가 있어도 웹 브라우저는 th: 속성을 
	    알지 못하므로 무시한다. 
	  - 따라서 HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다. 
	- URL 링크 표현식 - @{...}
	  - th:href="@{/css/bootstrap.min.css}"
	    - @{...}: 타임리프는 URL 링크를 사용하는 경우 @{...}를 사용한다. 이것을 
		  URL 링크 표현식이라 한다. 
		- URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다. 

  상품 등록 폼으로 이동 
    - 속성 변경 - th:onclick
	  - onclick="location.href='addForm.html'"
	  - th:onclick="|location.href='@{/basic/items/add}'|"
	    여기에는 다음에 설명하는 리터럴 대체 문법이 사용되었다. 자세히 알아보자. 
	
	- 리터럴 대체 - |...|
	  - |...|: 이렇게 사용한다 
	    - 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다. 
	      - <span th:text="'Welcome to our application, ' + ${user.name} + '!'">
	    - 다음과 같이 리터럴 대체 문법을 사용하면 더하기 없이 편리하게 사용할 수 있다. 
		  - <span th:text="|Welcome to our application, ${user.name}!|">
		- 결과를 다음과 같이 만들어야 하는데 
		  - location.href='/basic/items/add'
		- 그냥 사용하면 문자와 표현식을 각각 따로 더해서 사용해야 하므로 다음과 같이 복잡해진다. 
		  - th:onclick="'location.href=' + '\'' + @{/basic/items/add} + '\''"
		- 리터럴 대체 문법을 사용하면 다음과 같이 편리하게 사용할 수 있다. 
		  - th:onclick="|location.href='@{/basic/items/add}'|"
	
	- 반복 출력 - th:each
	  - <tr th:each="item : ${items}">
	  - 반복은 th:each를 사용한다. 이렇게 하면 모델에 포함된 items 컬렉션 데이터가 item 변수에 
	    하나씩 포함되고, 반복문 안에서 item 변수를 사용할 수 있다. 
	  - 컬렉션의 수 만큼 <tr>..</tr>이 하위 태그를 포함해서 생성된다. 
	
	- 변수 표현식 - ${...}
	  - <td th:text="${item.price}">10000</td>
	  - 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다. 
	  - 프로퍼티 접근법을 사용한다. (item.getPrice())
	
	- 내용 변경 - th:text 
	  - <td th:text="${item.price}">10000</td>
	  - 내용의 값을 th:text의 값으로 변경한다. 
	  - 여기서는 10000을 ${item.price}의 값으로 변경한다. 
	
	- URL 링크 표현식2 - @{...}
	  - th:href="@{/basic/items/{itemId}(itemId=${item.id})}"
	  - 상품 ID를 선택하는 링크를 확인해보자. 
	  - URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다. 
	  - 경로 변수 ({itemId})뿐만 아니라 쿼리 파라미터도 생성한다. 
	  - 예) th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"
	    - 생성 링크: http://localhost:8080/basic/items/1?query=test
		
	- URL 링크 간단히 
	  - th:href="@{|/basic/items/${item.id}|}"
	  - 상품 이름을 선택하는 링크를 확인해보자.ㅏㅏ 
	  - 리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다. 

	참고 
	  - 타임리프는 순수 HTML 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 
	    뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다. JSP를 생각해보면, JSP 파일은 
		웹 브라우저에서 그냥 열면 JSP 소스코드와 HTML이 뒤죽박죽 되어 있어서 정상적인 확인이 
		불가능하다. 오직 서버를 통해서 JSP를 열어야한다. 
	  - 이렇게 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 
	    네츄럴 템플릿(natural templates)이라 한다. 
	
```


