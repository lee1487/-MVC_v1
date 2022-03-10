#스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 by 인프런 김영한 

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
-----------------------------------
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