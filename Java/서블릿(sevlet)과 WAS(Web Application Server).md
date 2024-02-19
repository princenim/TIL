# 📌 서블릿(servlet)과 WAS(Web Application Server)

서블릿과 WAS를 보기전에 먼저 동적 웹 페이지가 등장한 배결을 알고가자.

### 웹의 발전

1. 초기에는 정적인 HTML을 제공하는 웹 서버가 주를 이뤘다.
2. 웹의 발전에 따라 동적인 컨텐츠가 필요해고 이를 위해 CGI 등의 기술이 등장했다. CGI는 웹 서버와 외부 프로그램 간의 통신을 위한 인터페이스로 사용되었다.
3. 자바에서는 `서블릿`이 등장해 동적인 웹 어플리케이션을 개발하는데 사용되었다.
4. `서블릿`과 `JSP`를 실행하고 관리하는데 특화된 서버인 `WAS`가 등장했다. `WAS`는 웹 서버의 확장으로 동작하며 서블릿 컨테이너, JSP 엔진, 데이터 베이스 연동 기능등을 제공한다.
5. 자바 기반의 웹 어플리케이션 개발을 위한 표준화된 스펙인 `Java EE`가 등장했다. 이를 통해 다양한 기술과 API(서블릿, JSP ) 등을 사용해 기업급 어플리케이션을 개발할 수 있게 되었다
6. `Java EE`를 기반으로 하는 다양한 웹 기반 프레임워크가 등장했다. 스프링 프레임워크는 그중 하나이다.

### 자바 SE와 자바 EE

- `자바 SE` 즉 자바 언어라고 하는 대부분의 패키지가 포함된 에디션을 말한다.
- `자바 EE( Enterprise Edition)`은 자바 SE 플랫폼을 기반으로 그 위에 탑재한다. 웹 프로그래밍에 필요한 기능을 다수 포함한다 JSP, 서블릿 등을 포함한다.

# 1. 서블릿(servlet)

**서블릿은 자바를 사용해 웹 페이지를 동적으로 생상할 수 있게 만들어주는 자바 클래스이다.** 더 간단히 말하면 자바를 사용하여 웹 페이지를 동적으로 생성할 수 있게 만들어준다.

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

### 서블릿의 동작방식

![sevelet](https://github.com/princenim/TIL/assets/59499600/becb61c7-d8e2-4d37-ad2f-e3e502e43dea)

1. 클라이언트가 url을 입력하면 `http request`가 서블릿 컨테이너로 전송한다.
2. 요청을 받은 서블릿 컨테이너는 `HttpServletRequest,` `HttpServletResponse` 객체를 생성한다.
3. web.xml을 기반으로 사용자가 요청한 url이 어느 서블릿에 대한 요청인지 찾는다.
4. 해당 서블릿에서 `service()` 메소드를 호출한 후 어느 서블릿에 대한 요청인지 찾릿는다.
5. `doGet()` 이나 `doPost()` 메소드는 동적 페이지를 생성한 후 `HttpServletResponse`객체에 응답을 보낸다.
6. 응답이 끝나면 `HttpServletRequest`, `HttpServletResponse` 객체를 소멸시킨다.

# 2. 서블릿 컨테이너/웹 컨테이너

**서블릿 컨테이너는 서블릿을 관리해주는 컨테이너이다.** 즉 `JSP`와 `서블릿`을 실행시킬수 있는 소프트웨어를 말한다. 서버에 서블릿을 만들었다고 스스로 작동하는 것이 아니라 서블릿을 관리해주는것이 필요한데 이 역할을 해주는 것이 `서블릿 컨테이너`이다.

# 3. JSP (Java server page)

서블릿을 보다 편하게 사용할 수 있게 만들어진 기술이 JSP이다. 서블릿은 자바 소스코드속에 HTML 코드가 들어가 있는 형태이지만 JSP는 이와 반대로 HTML코드 속에 자바 소스코드가 들어가 있는 구조이다.

![jsp](https://github.com/princenim/TIL/assets/59499600/e1c80322-c906-48b5-ab82-f2839a08f724)

jsp 처리 과정이다. 두 가지의 경우가 있다.

- **JSP에 해당하는 서블릿이 존재하지 않을 경우**
1. 클라이언트 요청이 들어오면 JSP 페이지로부터 자바 코드를 생성한다 `.java` 파일을 말한다.
2. 이 자바 코드를 컴파일해서 서블릿 클래스를 생성한다. .`class`파일을 말한다.
3. 서블릿에 클라이언트 요청을 전달한다.
4. 서블릿이 요청을 처리한 결과를 응답으로 생성한다.
5. 응답을 웹 브라우저에 전송한다.
- **JSP에 해당하는 서블릿이 존재하는 경우**
1. 서블릿에 클라이언트 요청을 전달한다.
2. 서블릿이 요청을 처리한 결과를 응답으로 생성한다.
3. 응답을 웹 브라우저에 전송한다.

즉, JSP 페이지를 요청할 때에는 JSP를 직접 실행하는 것이 아니라 JSP를 자바 소스 코드(`.java`) 로 변환한 뒤 컴파일해서 생성한 서블릿을 실행하는 것이다.

# 4. WAS(Web Appliacation Server)

웹은 태생적으로 논문을 쉽게 읽기위해 개발한 기술이다. 그래서 사용자가 요청한 파일을 그대로 전달하는 서버로 사용했다. 이를 `웹 서버`라고  대표적인 웹 서버에는 `Apache`, `Nginx`가 있다. 한다. 
후에 웹이 발달하여 상황에 따라 동적으로 바뀌는 문서를 제공해야했고 이런 기능을 제공하는 서버를 `WAS`라고 한다.

![was3](https://github.com/princenim/TIL/assets/59499600/7ccc6d49-a5fe-4de1-b083-53a952d13da7)

`WAS`는  `Web Appliacation Server`의 약자로 `웹 서버 + 웹 컨테이너`를 가진 서버라고 보면 된다.  웹 컨테이너는 `서블릿`과 `JSP` 구동환경을 제공해주기 때문에 `서블릿 컨테이너`라고도 부른다.

![was2](https://github.com/princenim/TIL/assets/59499600/9be1dd0c-a4dd-4be0-84a5-3ab0f1e15b6b)

대표적인 WAS의 종류에는 `apache tomcat`, `JBOSS`이 있다.

`Apache Tomcat`은 자바 EE의 일부 기술을 지원한다. 기본적으로 `Apache Tomcat`은 `서블릿 API` , `JSP(JavaServer Pages)`를 지원한다. 즉 자바에서 `Apache Tomcat`을 사용하여 서블릿을 실행할 수 있다는 말이다. 웹 컨테이너의 유무로 웹 서버와 WAS를 나눌 수 있다. 따라서 톰캣에는 웹 컨테이너가 있으니 WAS이다.

## 4.1 WAS가 필요한 이유

웹 페이지에는 정적 컨텐츠(HTML,이미지)와 동적컨텐츠가 존재한다. 이때 웹 서버만을 사용한다면 사용자가 원하는 요청에 맞게 결과값을 모두 미리 만들어 놔야한다 따라서 웹 서버가 모든 일을 처리할 수 없으며 WAS를 통해 그때 그때 결과를 만들어 결과를 제공함으로써 자원을 효율적으로 사용할 수 있다.