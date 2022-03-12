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
	  - application.propertiest에 
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


