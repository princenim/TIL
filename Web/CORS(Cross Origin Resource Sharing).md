# **📌** CORS(Cross Origin Resource Sharing)

# 1. Origin

먼저 `CORS` 을 알기전에 `Origin`을 알아보자. `Origin`이란 출처라는 뜻으로 `URL`에서 `프로토콜`, `도메인`, `포트번호`를 합친 부분을 의미한다.

즉,  `https://hazel.com:80/posts/123456?data=789#abc` 이런 URL이 존재한다면 `https://it-eldorado.com:80` 이 부분이 `Origin`이다.

# 2. SOP(Same Origin Policy)

`SOP`는 `동일 출처 정책`이라는 뜻으로 같은 출처의 리소스만 상호작용을 허용하는 정책이다. 이 정책은 기본적으로 웹 보안을 강화하기 위해 존재한다.

만약에 SOP가 없었다면 모든 출처에서 접근이 가능하고 외부서버가 악의적으로 접근해 조작할 가능성이 있다. 따라서 SOP는 같은 출처의 리소스에 대해서만 API요청을 가능하게 하고, 다른 출처의 리소스에 전급하기 위해서는 추가적인 매커니즘을 사용하여 접근할 수 있도록 하였다.

예를 들어 도메인 A에서 실행중인 코드는 다른 도메인 B에 위치한 리소스에 직접 접근할 수 있다.

# 3. CORS(Cross Origin Resource Sharing)

`SOP`로 인해 다른 출처의 자원을 사용할 수 없다. 이는 악의적인 요청을 막을 수 있다는 장점이 있지만 유효한 요청또한 막아버린다는 단점이있다. 웹 환경은 개방적이고 서로 다른 출처에서 리소스를 가져와서 사용하는 행위자체를 막아버리면 어플리케이션이 동작하기 힘들다. 따라서 출처가 다른 리소스를 사용할 수 있도록 하는 몇가지 예외 조항이 있는데 그중 하나가 바로 CORS 정책을 지킨 리소스 요청이다.

`CORS`는 `Cross Origin Resource Sharing`의 약자로 `교차 출처 리소스 공유`를 말한다.

동작 원리는 단순하다. 브라우저는 다른 `Origin` 으로 요청을 보낼 때 `Origin` 헤더에 자신의 `Origin`을 설정하고, 서버로부터 응답을 받으면 응답의 `Access-Control-Allow-Origin`헤더에 설정된 `Origin`의 목록에 요청의 `Origin` 헤더 값이 포함되어있는지 검사한다.

더 자세하게 설명하면 기본적으로 웹 어플리케이션에서 출처가 다른 곳에 요청을 할때 요청 헤더에 `Origin` 이라는 필드를 함께 보낸다. ( `https://www.google.com` → `https://my-server.com` 로 요청 )

```java
// Origin : https://www.google.com
```

예를 들어, 출처가 `https://my-server.com`인 서버에서 `https://www.google.com` 로 부터 오는 리소스 요청을 허용하고자 한다면, 응답 헤더에 `Access-Control-Allow-Origin: https://www.google.com` 을 설정하여 응답한다. 그리고 응답을 받은 브라우저는 자신이 보냈던 요청의 `Origin`과 서버 응답의 `Access-Control-Allow-Origin` 값이 일치하는지 확인하고 일치하면 출처가 달라도 유효한 요청이라고 판단해 받아온 리소스를 사용한다.

## 3.1 CORS 에러

`CORS 에러`는 브라우저에서 발생하는 오류로 `CORS` 정책 위반시에 나타난다. 이 에러는 프론트와 백엔드로 나누어 개발할 경우 흔히 나타나는데 Spring을 사용해 `CORS` 에러를 해결할 수 있다.

### **@CrossOrgin() 어노테이션 사용**

해당 어노테이션을 클래스나 메소드에 붙이면 해당 클래스와 메소드는 `CORS`를 허용한다.

### Spring Security 에서 CORS 설정

`Spring security`를 사용하면 모든 외부 요청에 대해 `CORS`를 `Block`한다 따라서 `CORS`에 대한 설정을 커스텀해야한다.  `SecurityFilterChain`을 스프링 빈으로 등록할때 `CORS`와 관련된 설정을 할 수 있다.

```java

//spring boot 3점대 
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.cors(corsCustomizer -> corsCustomizer.configurationSource(request -> {
            CorsConfiguration config = new CorsConfiguration();
            config.setAllowedOrigins(Arrays.asList(
                    "http://localhost:3000",
										"etc.."
            ));
            config.setAllowedMethods(Collections.singletonList("*"));
            config.setAllowCredentials(true);
            config.setAllowedHeaders(Collections.singletonList("*"));
            config.setMaxAge(3600L);
            return config;
        }));
}
```