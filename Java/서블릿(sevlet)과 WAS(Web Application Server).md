# 📌 서블릿(servlet)과 WAS(Web Application Server)

서블릿과 WAS를 보기전에 먼저 동적 웹 페이지가 등장한 배경을 알고가자.

### 웹의 발전

1. 초기에는 정적인 `HTML`을 제공하는 웹 서버가 주를 이뤘다.
2. 웹의 발전에 따라 동적인 컨텐츠가 필요해고 이를 위해 `CGI` 등의 기술이 등장했다. `CGI`는 웹 서버와 외부 프로그램 간의 통신을 위한 인터페이스로 사용되었다.
3. 자바에서는 `서블릿`이 등장해 동적인 웹 어플리케이션을 개발하는데 사용되었다. 서블릿는 자바 코드 안에  HTML 코드를 포함하는 형태로 구성되어있어 복잡한 웹 페이지를 작성하는 데 번거로움이 있었다.
4. 서블릿의 불편함으로 해소하기 위해 `JSP`가 등장했다. `JSP`는 `HTML`문서 안에 자바 코드를 삽입하는 형태로 작성되며 이렇게 작성된 JSP 페이지는 서블릿으로 변환되어 실행된다.
5. `서블릿`과 `JSP`를 실행하고 관리하는데 특화된 서버인 `WAS`가 등장했다. `WAS`는 웹 서버의 확장으로 동작하며 서블릿 컨테이너, JSP 엔진, 데이터 베이스 연동 기능등을 제공한다.
6. 자바 기반의 웹 어플리케이션 개발을 위한 표준화된 스펙인 `Java EE`가 등장했다. 이를 통해 다양한 기술과 API(서블릿, JSP) 등을 사용해 기업급 어플리케이션을 개발할 수 있게 되었다.
7. `Java EE`를 기반으로 하는 다양한 웹 기반 프레임워크가 등장했다. 스프링 프레임워크는 그중 하나이다.

### 자바 SE와 자바 EE

- `자바 SE` 는 흔히 자바 언어라고 부르는 대부분의 패키지가 포함된 에디션을 말한다.
- `자바 EE(Enterprise Edition)`은 자바 SE 플랫폼을 기반으로 그 위에 탑재한다. 좀 더 쉽게 말하면 자바를 이용한 서버 측 개발을 위한 플랫폼이다. 기업 환경에서 대규모 및 분산형 어플리케이션을 개발하고 운영하기 위한 다양한 기능과 API를 제공한다. `JSP`, `Servlet`, `JDBC` 등을 포함한다. 최근에는 Jakerta EE로 명칭이 변경되었다.

# 1. 서블릿(servlet)

서블릿은 자바를 사용해 웹 페이지를 동적으로 생성할 수 있게 만들어주는 자바 클래스이다. 서블릿은 자바 코드 안에 HTML 코드를 포함되어 작성되며 Java EE에서 제공하는 서블릿 컨테이너(e.g Apache Tomcat)에서 실행된다. (정확히 말하면 Tomcat은 Java EE의 서블릿 컨테이너를 제공하는 WAS이다.) 서블릿은 HTTP 서비스를 위한 doGet(), doPost()와 같은 메소드를 제공하며 이런 메소드를 통해 서블릿은 클라이언트의 GET, POST 요청에 대응한다.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // HTTP 응답의 Content Type을 설정합니다.
        response.setContentType("text/html");

        // 응답으로 보낼 내용을 작성합니다.
        response.getWriter().println("<html><body>");
        response.getWriter().println("<h1>Hello, Servlet!</h1>");
        response.getWriter().println("</body></html>");
    }
}
```

서블릿은`JAVA EE` 플랫폼의 일부로 제공되며 `javax.servlet` 패키지의 인터페이스와 클래스들을 사용하여 개발된다.


## 2.1 서블릿의 생명주기

서블릿의 생명주기는 크게 `로딩`, `객체 생성`,  `초기화(Initialization)`, `서비스(service)` , `소멸(Destruction)` 단계라고 볼수 있다. 더 자세히 보자.

### 1.로딩 (Loading)

서블릿에 최초의 요청이 들어올때  클래스 로더에 의해 서블릿 클래스가 메모리에 로딩된다. 서블릿 클래스가 로딩되면 해당 클래스의 클래스 메타데이터는 `JVM`의  `Method Area`에 저장된다.

웹 어플리케이션이 시작될 때 모든 서블릿 클래스를 미리 로딩하지 않고 실제로 요청이 들어올 때 동적으로 필요한 서블릿 클래스를 로딩한다(= `레이지 로딩`)  따라서 서버가 시작된다고 하더라도 서블릿이 생성되지 않았음을 의미한다. 따라서 보통 배포를 할때 `warm up` 이라는 과정을 한다. 서비스에서 쓰는 주요 URL 들을 실제로 배포 스크립트 상에서 임의로 한번 식 호출함으로써 미리 서블릿을 생성 및 초기화해 성능을 향상시키는 것이다.

### 2. 객체 생성

클래스가 메모리에 로딩된 후 서블릿 컨테이너는 해당 서블릿 클래스의 객체를 생성한다. 이때 서블릿 객체는 요청이 발생할 때마다 생성되는 것이 아니라 이미 객체가 생성된 상태라면 생성된 객체를 계속 사용한다. 이때 서블릿 객체는 JVM의 heap 영역에 생성된다.

### 3. 초기화(Initialization)

`init()` 메소드가 서블릿 초기화 작업을 수행한다. 이 메소드는 서블릿 컨테이너에 의해 일생 동안 단 한번 호출되어 실행된다.

```java
public void init(ServletConfig config) throws ServletException;
```

### 4. 서비스 (Service)

서블릿은 클라이언트로부터 요청을 처리하기 위해 매 요청마다 `service()`메소드를 호출한다. `HTTP 메소드`에 따라서 `doGet()`,`doPost()` 등을 호출한다. 그리고 각 요청에 대한 실제 로직을 이 단계에서 처리한다.

### 5. 소멸(**Destruction**)

서블릿이 더 이상 필요하지 않아 서블릿 컨테이너가 서블릿을 소멸하기 위해 `destroy()` 메소드를 호출한다. 이 메소드를 호출해 자원을 해제하고 정리한다. 이 메소드도 서블릿의 생명주기 동안 단 한번만 호출된다.

위의 내용을 한번 더 정리해보자면 다음과 같다.

![sevelet](https://github.com/princenim/TIL/assets/59499600/6022c105-424b-4f55-b3b1-aadbc814e8b4)

1. 클라이언트가 요청을 한다.
    1. `Servlet` 객체가 없다면  `Servlet` 클래스가 메모리에 로딩되어 Servlet 객체가 생성된다. 이때 `init()` 메소드를 호출해 `Servlet`을 초기화한다.
    2. `Sevlet` 객체가 있다면 바로 생성된 `Sevlet` 객체를 사용한다.
2. `service()` 메소드를 호출해 `Servlet`이 **브라우저의 요청을 처리한다.**
3. `service()` 메소드는 특정 HTTP 요청(GET, POST 등)을 처리하는 메서드 (`doGet(), doPost()` 등)를 호출한다.
4. 서버는 `destroy()` 메소드를 호출해 `Servlet`를 제거한다.

## 2.2 서블릿 컨테이너/웹 컨테이너

서블릿 컨테이너는 서블릿을 생명주기를 관리하고 서블릿을 실행하는 환경을 제공해주는 컨테이너이다. 즉  서블릿을 실행시킬수 있는 소프트웨어를 말한다.

# 3. JSP (Java server page)

서블렛을 보다 편하게 사용할 수 있게 만들어진 기술이 `JSP`이다. **서블렛은 자바 소스코드속에 HTML 코드가 들어가 있는 형태이지만 JSP는 이와 반대로 HTML코드 속에 자바 소스코드가 들어가 있는 구조**이다.

```bash
##jsp 

<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>간단한 JSP 예제</title>
</head>
<body>

<%
    // Java 코드 작성
    String greeting = "안녕하세요!";
    out.println("<h1>" + greeting + "</h1>");

    // 조건문 사용
    int number = 5;
    if (number > 0) {
        out.println("<p>양수입니다.</p>");
    } else if (number < 0) {
        out.println("<p>음수입니다.</p>");
    } else {
        out.println("<p>0입니다.</p>");
    }
%>

</body>
</html>
```

## 3.1 JSP 처리과정

![jsp](https://github.com/princenim/TIL/assets/59499600/f14c9d22-2eef-40c1-ae09-088c5da0b9c5)

다음은 JSP 처리 과정이다. 두 가지의 경우가 있다.

### **JSP에 해당하는 서블렛이 존재하지 않을 경우**

1. 클라이언트 요청이 들어오면 `JSP 페이지`로부터 자바 코드를 생성한다 `.java` 파일을 말한다.
2. 이 자바 코드를 컴파일해서 `서블릿 클래스`를 생성한다. `.class`파일을 말한다.
3. 서블릿에 클라이언트 요청을 전달한다.
4. 서블릿이 요청을 처리한 결과를 응답으로 생성한다.
5. 응답을 웹 브라우저에 전송한다.

### **JSP에 해당하는 서블릿이 존재하는 경우**

1. 서블릿에 클라이언트 요청을 전달한다.
2. 서블릿이 요청을 처리한 결과를 응답으로 생성한다.
3. 응답을 웹 브라우저에 전송한다.

즉, `JSP 페이지`를 요청할 때에는 `JSP` 를 직접 실행하는 것이 아니라 JSP를 자바 소스 코드(.java) 로 변환한 뒤 컴파일해서 생성한 서블릿을 실행하는 것이다.

# 4. WAS(Web Appliacation Server)

웹은 태생적으로 논문을 쉽게 읽기위해 개발한 기술이다. 그래서 사용자가 요청한 파일을 그대로 전달하는 서버로 사용했다. 이를 `웹 서버`라고  대표적인 웹 서버에는 `Apache`, `Nginx`가 있다. 한다. 
후에 웹이 발달하여 상황에 따라 동적으로 바뀌는 문서를 제공해야했고 이런 기능을 제공하는 서버를 `WAS`라고 한다.

![was3](https://github.com/princenim/TIL/assets/59499600/7ccc6d49-a5fe-4de1-b083-53a952d13da7)

`WAS`는  `Web Appliacation Server`의 약자로 `웹 서버 + 웹 컨테이너`를 가진 서버라고 보면 된다.  웹 컨테이너는 `서블릿`과 `JSP` 구동환경을 제공해주기 때문에 `서블릿 컨테이너`라고도 부른다.

![was2](https://github.com/princenim/TIL/assets/59499600/9be1dd0c-a4dd-4be0-84a5-3ab0f1e15b6b)

자바에서 `WAS`라고 하면 `Java EE` 표준을 준수하는 웹 어플리케이션 서버를 말한다. 대표적인 `WAS`의 종류에는 `apache tomcat`, `JBOSS`이 있다.

즉, `Apache Tomcat`은 자바 EE의 일부 기술을 지원한다. 기본적으로 `Apache Tomcat`은 `서블렛 API` , `JSP(JavaServer Pages)`를 지원한다. 즉 자바에서 `Apache Tomcat`을 사용하여 서블릿을 실행할 수 있다는 말이다. 웹 컨테이너의 유무로 웹 서버와 WAS를 나눌 수 있는데 톰캣에는 웹 컨테이너가 있으니 WAS이다.



## 4.1 WAS가 필요한 이유

웹 페이지에는 정적 컨텐츠(HTML,이미지)와 동적컨텐츠가 존재한다. 이때 웹 서버만을 사용한다면 사용자가 원하는 요청에 맞게 결과값을 모두 미리 만들어 놔야한다 따라서 웹 서버가 모든 일을 처리할 수 없으며 WAS를 통해 그때 그때 결과를 만들어 결과를 제공함으로써 자원을 효율적으로 사용할 수 있다.