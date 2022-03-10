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