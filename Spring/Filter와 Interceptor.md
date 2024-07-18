
# 📌 Filter와 Interceptor

필터와 인터셉터 모두 공통 관심 사항(로직에서 공통으로 관심이 있는것)을 처리하는 역할을 한다. 이런 공통 관심사는 스프링의 AOP로도 해결할 수 있지만 필터 또는 인터셉터를 사용하는 것이 좋다. **웹과 관련된 관심사를 처리할때는 HTTP 헤더나 URL 정보들이 필요**한데, 서블릿 필터나 스프링 인터셉터는 `HTTPServletRequest`를 제공한다. 

# 1. Filter

서블릿 필터는 `서블릿`이 지원하는 수문장의 역할을 한다.

### 필터 흐름

```java
필터 흐름 :  HTTP 요청 → WAS → 필터 → 서블릿(디스패처 서블릿) → 컨트롤러
```

필터를 적용하면 필터가 호출된 다음에 서블릿이 호출된다. 그래서 모든 고객의 요청 로그를 남기는 요구사항이 있다면 필터를 사용할 수 있다. 필터는 특정 URL패턴에 적용할 수 있다.

### 필터 제한

```java
필터 흐름 :  HTTP 요청 → WAS → 필터 → 서블릿(디스패처 서블릿) → 컨트롤러 //로그인 사용자 
필터 흐름 :  HTTP 요청 → WAS → 필터(적절하지 않은 요청이라 판단, 서블릿 호출) //비 로그인 사용자 
```

필터에서 적절하지 요청이라고 판단다면 거기서 끝을 낼 수 있다.

### 필터 체인

```java
필터 흐름 : HTTP 요청 → WAS → 필터1 → 필터2 -> 필터3 -> 서블릿(디스패처 서블릿) → 컨트롤러
```

필터는 체인으로 구성되는데 중간에 필터를 자유롭게 추가할 수 있다. 예를들어 로그를 남기는 필터, 그리고 로그인 여부를 체크하는 필터를 만들 수 있다.

### 필터 인터페이스

```java
package jakarta.servlet;

public interface Filter {

  
    default void init(FilterConfig filterConfig) throws ServletException {
    }
    
    void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException;

    default void destroy() {
    }
}
```

필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 `싱글톤` 객체로 생성하고 관리한다.

- `init()` : 필터 초기화 메서드. 서블릿 컨테이너가 생성될 때 호출된다.
- `doFileter()`: 고객의 요청이 들어올 때마다 해당 메소드가 호출된다. 이 메소드를 구현하면 된다.
- `destroy()` : 필터 종료 메서드로 서블릿 컨테이너 종료 시 호출된다.

### 필터 사용 예시

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class LogFilter implements Filter { //인터페이스 구현

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("log filter doFilter");

        HttpServletRequest httpRequest = (HttpServletRequest) request; //ServletRequest -> HttpServeltRequest로 다운캐스팅
        String requestURI = httpRequest.getRequestURI();

        String uuid = UUID.randomUUID().toString(); //요청을 구분하기 위함

        try {
            log.info("REQUEST [{}][{}]", uuid, requestURI);
            chain.doFilter(request, response);//다음 필터 호출 (있으면 필터호출, 없으면 서블릿)
            //다음 로직이 다 끝난 다음에 밑으로 내려감
        } catch (Exception e) {
            throw e;
        } finally { //무조건 실행
            log.info("RESPONSE [{}][{}]", uuid, requestURI);
        }

    }

    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}

```

```java
import javax.servlet.Filter;
import java.util.List;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean //필터 만든다음에 필터를 등록해아함.
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter()); //만든 필터
        filterRegistrationBean.setOrder(1); //필터의 순서를 정해줌.
        filterRegistrationBean.addUrlPatterns("/*"); //어떤 패턴을 사용할건지

        return filterRegistrationBean;
    }
}

```

필터를 만든 후에 등록한다. 필터를 등록하는 방법은 여러가지지만 `FilterRegistrationBean` 을 사용해 등록할 수 있다.

# 2. Interceptor

인터셉처도 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술이다. 필터가 서블릿이 제공하는 기술이라면 인터셉터는 스프링 MVC가 제공하는 기술이다.

### 인터셉터 흐름

```java
필터 흐름 :  HTTP 요청 → WAS → 필터 → 서블릿(디스패처 서블릿) → 스프링 인터셉터 → 컨트롤러
```

스프링 인터셉터는 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출 된다. 스프링 인터셉터에도 URL 패턴을 적용할 수 있는데 서블릿 URL 패턴보다 매우 정밀하게 설정할 수 있다.

### 인터셉터 제한

```java
필터 흐름 :  HTTP 요청 → WAS → 필터 → 서블릿(디스패처 서블릿) → 스프링 인터셉터 → 컨트롤러 //로그인 사용자
필터 흐름 :  HTTP 요청 → WAS → 필터 → 서블릿(디스패처 서블릿) → 스프링 인터셉터(적절하지 않은 요청이라 판단, 컨트롤러 호출X) // 비로그인사 사용자
```

### 인터셉터 체인

```java
필터 흐름 :  HTTP 요청 → WAS → 필터 → 서블릿(디스패처 서블릿) → 스프링 인터셉터1 → 스프링 인터셉터2→ 컨트롤러
```

필터와 같이 체인을 걸 수 있다.

### 인터셉터 인터페이스

스프링의 인터셉터를 사용하려면 `HandlerInterceptor` 를 사용하면 된다.

```java

package org.springframework.web.servlet; //spring MVC가 제공하는 패키지

public interface HandlerInterceptor {

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}

}
```

- 단순히 doFilter을 제공하는 필터와는 다르게 인터셉터는 컨트롤러 호출 전(`preHandle`), 호출 후(`postHandle`) , 요청 완료 이후(`afterCompletion`)와 같이 단계적으로 세분화되어있다.
- **그리고 서블릿 필터의 경우 단순히 `request`, `response`만 제공했지만, 인터셉터는 `handler`가 호출되는지 호출 정보도 받을 수 있다. 이 handler는  실행하려는 메소드 즉 현재 실행하려는 컨트롤러를 말한다.  그리고 어떤 `ModelAndView` 가 반환되는지 응답 정보도 받을 수 있다.**

### 인터셉터 호출 흐름

<img width="548" alt="inter" src="https://github.com/user-attachments/assets/a89ce929-baf2-47b8-aa8a-5ba80e1b56e0">

- `preHandle` : 컨트롤러 호출 전에 호출된다. (정확히는 핸들러 어댑터 호출 전에 호출된다. 이때 응답값true면 다음으로 진행하고 false면 진행하지 않는다.
- `postHandle`: 컨트롤로 호출 후에 호출된다(정확히는 핸들러 어댑터 호출 후에 호출된다.)
- `afterCompletion` : 뷰가 렌더링 된 이후에 호출된다.

그리고 좀더 정확히 말하면 `HandlerMapping`은 URL과 매칭되는 Contoroller를 선택해주는 역할을 한다. 그리고 HandlerAdaptor는 Controller의 메소드를 실행해주는 역할을 한다.

1. `“/test”`라는 요청이 들어온다
2. `DispatcherServlet`이 `HanderMapping`을 통해 `“/test”` 라는 요청에 해당하는 Controller를 선택한다.
3. 요청을 인터셉터의 `preHandle` 메소드가 가로채 호출한다.
4. `HandlerAdaptor`를 통해서 `“/test”` 에 해당하는 Controller를 실행한다.
5. `postHandle` 이 실행되고 view가 실행되어 response를 클라이언트에게 전달한다.
6. `afterCompletion` 가 view 실행 이후에 동작한다. (항상 호출)

### 인터셉터 예외 상황

<img width="558" alt="inter2" src="https://github.com/user-attachments/assets/df655c57-0bd7-47b1-92ab-803f7d618a7d">

**예외 발생 시**

- `preHandle` : 컨트롤러 호출 전에 호출된다.
- `postHandle`:  컨트롤러에서 예외가 발생하면 `postHandle` 은 호출되지 않는다.
- `afterCompletion` : `afterCompletion` 는 항상 호출된다. 이 경우 예외를 파라미터로 받아서 어떤 예외가 발생했는지 로그로 출력할 수 있다.

따라서 예외가 발생했을 때 `postHandle` 은 호출되지 않으므로 예외와 무관하게 공통 처리를 하려면 `afterCompletion` 을 사용해야한다.

즉, 인터셉터는 스프링 MVC 구조에 특화된 필터 기능을 제공하고 특별히 필터를 사용해야하는 상황이 아니라면 인터셉터를 사용하는 것이 더 편리하다.

### 인터셉터 사용 예시

```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    //handler는 수행할 controller를 말한다. 

        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        request.setAttribute(LOG_ID, uuid); //uuid를 afterCompletion에 넘기기 위함.

        //@RequestMapping: HandlerMethod
        //정적 리소스: ResourceHttpRequestHandler
        if (handler instanceof HandlerMethod) {
            HandlerMethod hm = (HandlerMethod) handler; //호출할 컨트롤러 메서드의 모든 정보가 포함되어 있다.
        }

        log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler); //어떤
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        String logId = (String) request.getAttribute(LOG_ID);
        log.info("RESPONSE [{}][{}][{}]", logId, requestURI, handler);
        if (ex != null) {
            log.error("afterCompletion error!!", ex);
        }

    }
}
```

```java
@Configuration //인터셉터 등록
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1) //순서
                .addPathPatterns("/**") //하위 경로 다
                .excludePathPatterns("/css/**", "/*.ico", "/error"); //이 경로는 제외
    }
}
```

그리고 인터셉터의 URL 경로는 필터에서 제공하는 URL경로보다 더 정밀하게 설정할 수 있다.

# 3. 필터와 인터셉터 차이점

|  | 필터 | 인터셉터 |
| --- | --- | --- |
| 관리되는 컨테이너 | 서블릿 컨테이너 | 스프링 컨테이너 |
| 스프링의 예외처리 여부 | X | O |
| Request/Response 객체 조작 가능 여부 | O | X |

### 스프링의 예외 처리 여부

일반적으로 스프링은 `ControllerAdvice`와 `ExceptionHandler`를 이용한 예외 처리 기능을 주로 사용한다.  만약 로직에서 `MemberNotFoundException`를 던져 404로 응답을 반환하길 원한다면 다음과 같이 구현할 것이고, 해당 예외가 발생했을 때 예외는 서블릿까지 전달되지 않고 `GlobalExceptionHandler`  내에서 처리어 클라이언트에게 반환된다.

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(MemberNotFoundException.class)
    public ResponseEntity<Object> handleMyException(MemberNotFoundException e) {
        return ResponseEntity.notFound().build();
    }
    ...
}
```

하지만 필터에서 발생한 예외는 스프링 컨텍스트에 도달하지 않고 서블릿 레벨에서 처리된다. 따라서 필터에 던져진 예외는 위의 `@RestControllerAdvice` 에 의해 처리되지 않고, 결과적으로 예외를 핸들링하지 못해 500 Internal Server Error를 반환한다. 하지만 인터셉터는 스프링 컨테이너에 의해 수행되기 때문에 예외가 캐치된다.

```java
public class MyFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletResponse servletResponse = (HttpServletResponse) response;
        servletResponse.setStatus(HttpServletResponse.SC_NOT_FOUND);
        servletResponse.getWriter().print("Member Not Found");
    }
}
```

### Request/Response 객체 조작 가능 여부

필터는 Request/Response를 조작할 수 있지만, 인터셉터는 조작할 수 없다. 여기서 조작이란 다른 객체로 바꾼다는 이야기이다.

```java
public MyFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        //다음 필터 호출 할때 개발자가 다른 request와 response를 넣어줄 수 있음
        chain.doFilter(new MockHttpServletRequest(), new MockHttpServletResponse());       
    }
    
}
```

```java
public class MyInterceptor implements HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // Request/Response를 교체할 수 없고 boolean 값만 반환할 수 있다.
        return true;
    }
}
```

코드를 보면 필터는 개발자가 다른 request, response를 넣어줄 수 있는 반면에 인터셉터는 true를 반환하면 다음 인터셉터가 호출되거나 컨트롤러로 요청이 전달되고, false라면 컨트롤러 요청이 중단된다. 그리고 Request, Response 객체를 넘겨줄 수도 없다.

# 4. 필터와 인터셉터의 데이터 접근 범위

필터의 경우 `ServletRequest`,  `ServletResponse` 를 가진다. 그리고 인터셉터는 `ServletRequest` , `ServletResponse` 에 추가로 `handler` 를 가진다. `handler`는 요청을 처리한 실제 컨트롤러와 관련된 정보를 포함한다. 일반적으로 컨트롤러 클래스의 메소드와 매핑되어있다. 따라서 보통 메소드 이름, 메소드의 어노테이션, 메소드 매개변수 등의 정보를 갖고있다.

### ServletRequest

`ServletRequest` 은 모든 서블릿 요청의 기본 인터페이스로 클라이언트의 요청 정보를 서블릿으로 넘겨주기 위한 객체이다. 요청이 들어오면 서블릿 컨테이너가 `ServletRequest` 를 생성하고 이 객체를 서블릿의 service의 파라미터로 전달한다. FTP, SMTP등 다른 프로토콜을 사용하는 서블릿에서도 사용할 수 있다.

[ServletRequest](https://javaee.github.io/javaee-spec/javadocs/javax/servlet/ServletRequest.html)

### HttpServletRequest

`HttpServletRequest`는 `ServletRequest` 를 확장한 인터페이스로 HTTP 요청에 특화된 메소드와 기능을 추가한 인터페이스이다. `getHeader`와 `getParameter` , `getMethod`와 같은 HTTP 특화 메소드를 제공한다.

[HttpServletRequest](https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpServletRequest.html)

### ServletResponse

`ServletResponse` 는 응답을 생성하고 클라이언트로 전송하는데 사용하는 인터페이스로 데이터를 처리하고 응답을 구성하는데 사용한다. ServletResponse를 통해서 HTML페이지, JSON데이터를 생성할 수 있다.

### **HttpServletResponse**

`HttpServletResponse`는 `ServletResponse`을 상속한 인터페이스로 HTTP프로토콜에 특화된 기능을 제공한다. 상태 코드를 설정하거나, 헤더설정, 쿠키관리와 같은 기능을 갖고있다.