# 📌 Spring MVC의 요청 처리 과정

먼저 Spring MVC를 알기전에 `MVC 패턴`을 알아야한다.

# 1. MVC 패턴

`MVC 패턴`은 어플리케이션을 개발할 때 사용하는 `디자인 패턴`이다. 어플리케이션의 개발 영역으로 `MVC(Model-View-Contorller)`로 나누어서 개발한다.

`사용자가 보는 페이지`, `데이터 처리`, `중간에서 제어하는 컨트롤`로 구분함으로써 각각 맡은 역할에만 집중을 할 수 있으므로 시스템결합도를 낮추고, 중복코드를 제거할 수 있다.

### Model

모델은 어플리케이션의 상태나 데이터를 나타낸다. 일반적으로 DB에서 데이터를 가져오거나 데이터를 변경하는 로직을 포함한다.  모델은 view나 controller에 독립적이다.

### View

사용자에게 정보를 표시하고, 사용자 입력을 받아 컨드롤러에 전달한다.

### Controller

컨트롤러는 클라이언트 측의 요청을 직접적으로 전달받는 엔드포인트로 Model과 View의 중간에서 상호작용을 해주는 역할을 한다.

## 1.1 MVC1, MVC2

`MVC 패턴`은 크게 `MVC1` 패턴과 `MVC2`패턴으로 나눌 수 있다.

### MVC 1 패턴

![mvc1](https://github.com/princenim/TIL/assets/59499600/965d3705-089f-4cee-a4fd-263c9551e8ed)

사용자로부터 요청이 들어오면 DB로부터 필요한 데이터를 받은 Model 객체를 JSP 페이지(view)에 담아 응답으로 보내는 형태이다. `MVC1` 패턴에서는 `View`와 `Controller`의 역할이 합쳐져있다. `JSP`에서 두 역할을 담당한다. 따라서 `JSP` 페에지 내에 너무 많은 코드가 들어가게 되고, 코드의 가독성이 떨어짐과 동시에 코드가 복잡해질 가능성이 있다. 이러한 점을 보완하여 `MVC2` 패턴이 등장했다.

### MVC2 패턴

![mvc2](https://github.com/princenim/TIL/assets/59499600/d867a4e8-0ec3-4793-b0ce-b7d036c7f027)

`MVC2` 패턴은 `MVC1` 패턴을 보완한 패턴이다. 기존에 뷰와 컨트롤러 역할을 모두 수행하던 `JSP`는 뷰의 역할만 수행하고, 컨트롤러는 `Servlet`이 수행한다. 따라서 `Servlet`이 비즈니스 로직을 수행하고 모델을 호출해 데이터를 요청하면 최종적으로 뷰 역할을 하는 `JSP`가 화명을 출력한다. 따라서 `HTML` 과 `Java` 코드가 분리되어 유지보수가 수월해진다.

# 2. Spring MVC

`Spring MVC`는 `Spring Framework`에서 제공하는 웹 어플리케이션 개발을 위한 모듈 중 하나이다. 이는 `MVC 패턴`을 기반으로 하며 Spring Framework의 DI, IoC를 활용해 객체간의 의존성을 관리하고 제어역전을 지원한다.

## 2.1 Spring MVC의 구조

먼저 Spring MVC의 구조를 알아보자.

![spring mvc ](https://github.com/princenim/TIL/assets/59499600/b7ff23e4-7041-4a63-97bc-218eecf452cf)

### DispatcherServlet

클라이언트의 요청에 전달받아 컨트롤러가 리턴한 결과값을 View에 전달해 알맞은 응답을 생성한다.

`Spring MVC`에서  `Front Controller` 는 `DispatcherServlet`이다. `Front Controller`란 `controller` 앞에서 클라이언트의 요청을 가장 먼저 받아들이고 이를 처리한다. 따라서 `DispatcherServlet`은 앞에서 요청을 처리하고 이를 처리할 적절한 핸들러에게 전달한다.

### Handler Mapping

`Handler Mapping`은 클라이언트의 요청 `url`을 어느 `controller`가 처리할지 결정한다. 요청을 받았을 때 들어온 `HTTP 메소드`와 `URL`을 해석해서 어느 `Controller`로 전달할지 결정한다.

### Handler Adapter

`Handler Mapping`을 통해 어느 `controller`가 처리할지 결정하면 `Handler Adapter`가 실제로 `controller`를 호출하고 실행한다. 그리고 그 결과를 적절한 형태로 반환한다.

### Controller (Handler)

클라이언트 요청을 처리하고 비즈니스 로직를 수행한다. `@Controller` 어노테이션을 사용하여 표시되며 요청을 받아 비즈니스 로직을 수행 후 적절한 응답을 생성해 반환한다.

### View Resolver

`controller`가 반환하는 뷰 이름을 기반으로 실제 뷰 객체를 찾아내고 이를 `DispatcherServlet` 에 반환한다. 예를들어 `controller`가 “`home`” 이라는 뷰 이름을 반환하면 `View Resolver`는 실제 “`home`”이라는 뷰 파일을 찾아서 뷰 객체로 변환하다. 이때 파일은 주로 `HTML`이며 `JSP`, `Thymeleaf` 등의 템플릿 엔진으로 처리된다.

단, `JSON API`형태의 응답을 반환할때는 `View Resolve` 를 사용하지 않고 controller (front controller X)가 직접 JSON 데이터를 반환한다.

## 2.2 Spring 의 요청 처리 과정

![spring](https://github.com/princenim/TIL/assets/59499600/07bfc1e2-a1d7-4150-8acb-587c7cd3abf8)

![springgg](https://github.com/princenim/TIL/assets/59499600/f0285048-6caa-4c91-bb08-7d2685321e37)

1. `HTTP` 요청이 서버에 도착
2. `Tomcat` 과 같은 `WAS`가 요청을 받고 해당 요청을 처리할 `서블릿 컨테이너`로 전달한다.
3. `Filter`가 요청을 가로채고 처리한다. 이 `Filter`은 `DispatcherServlet` 전후로 실행된다.
4. 필터의 처리가 완료되면 요청은 `DispatcherServlet`로 전달된다.
5. `DispatcherServlet`는 `HandlerMapping`을 통해 요청을 처리할 적절한 `Handler(Controller)`를 선택한다.
6. `Handler(Controller)`가 실행되기 전에, `Interceptor`가 요청을 가로채고 추가 작업을 수행한다.
7. `Controller`가 실행되고 `비즈니스 로직(service- repository)`을 수행한다. 그리고 결과를
8. 수행된 결과를 `DispatcherServlet`에 전달한다.  전달하기 전에 `Interceptor` 를 한 번 더 만난다.
9.  `DispatcherServlet`은 `Controller`에게 전달받은 `View` 이름과 매칭되는 실제 `view`파일을 찾기 위해 `View Resolver`을 요청한다.
10. `View Resolver`는 사용자에게 결과를 보여줄 `view`를 결정하고, 해당 객체를 `DispatcherServlet` 에게 전달한다.
11. `DispatcherServlet`은 반환된 뷰 객체를 실행해 클라이언트에게 보여줄 화면을 생성한다.

## Filter

`Filter`은 `스프링`의 기능이 아니라 자`바 서블릿`에서 제공하는 기능이다. `Filter`는 `DispatcherServlet` 의 전, 후로 동작한다. 그리고 `Filter`은 `FilterChain`을 통해 여러 `Filter`가 연쇄적으로 동작할 수있다.

필터는 주로 요청에 대한 인증, 권한 체크를 하는데 사용한다. 요청이 `DispatcherServlet` 에 들어오기 전에 헤더를 검사해 인증 토큰이 있는지 올바른지 확인할 수 있다.

## Interceptor

`Interceptor` 는 `controller`에 들어오는 요청 `HttpRequest`와 `controller`가 응답하는 `HttpResponse`를 가로채는 역할을 한다. `DispatcherServlet`  이후에 동작한다.

`Filter`과의 가장 큰 차이점은 `DispatcherServlet` 후 `HandlerMapping` 이 적절한 `controller`을 찾고 후에 동작하기 때문에 `Interceptor`는 `controller`에 직접 접근이 가능하다는 것이다. 이를 통해 `controller`에 전달되는 데이터를 수정하거나 특정 조건에 따라 `controller`를 실행하지 않을 수 있다. 하지만 `Filter`는 서블릿 컨테이너 수준에서 요청과 응답을 가로채기 때문에 `controller`에 직접적으로 접근할 수없다.

참고로 `Filter`와 `Interceptor`는 요청이 들어올때 와 나갈때 두번 작업을 수행할 수 있다.








**출처**
https://gowoonsori.com/blog/spring/architecture/