# 📌CORS(Cross Origin Resource Sharing)

# 1. Origin

먼저 `CORS` 을 알기전에 `Origin`을 알아보자. `Origin`이란 출처라는 뜻으로 `URL`에서 `프로토콜`, `도메인`, `포트번호`를 합친 부분을 의미한다.

`https://hazel.com:80/posts/123456?data=789#abc` 이런 URL이 존재한다면 `https://it-eldorado.com:80` 이 부분이 `Origin`이다.

# 2. SOP(Same Origin Policy)

`SOP`는 `동일 출처 정책`이라는 뜻으로 같은 출처에서 로드된 리소스만 접근할 수 있도록 하는 정책이다. 쉽게 말하면 내 웹 페이지에서 다른 웹페이지의 스크립트를 실행시키는 것을 막는 것을 말한다. 내가 네이버 블로그에 글을 썼을 때 다른 사이트에 있는 스크립트를 읽지 못하게 막는것을 말한다. 여기서 리소스(자원)이란 CSS나 이미지처럼 정적인 자원을 리소스라고 표현한다.

`SOP`의 주 목적은 보안 강화이다. 같은 출처의 리소스만 상호작용할 수 있도록 함으로써 악의적인 사이트에서 웹 페이지의 스크립트를 실행시켜 조작하거나 사용자 정보를 탈취하는것을 막아준다. `SOP`를 완화하여 다른 출처간의 리소스에 접근 가능하게 해주는 정책이 `CORS`이다.

# 3. CORS(Cross Origin Resource Sharing)

`CORS`는 `Cross Origin Resource Sharing`의 약자로 `교차 출처 리소스 공유`를 말한다. 즉, **다른 출처의 리소스에 접근하기 위한 보안 메커니즘을 말한다.**

`SOP`로 인해 다른 출처의 자원을 사용할 수 없는데 이는 악의적인 요청을 막을 수 있다는 장점이 있지만 유효한 요청 또한 막아버린다는 단점이있다. 웹 환경은 개방적이고 서로 다른 출처에서 리소스를 가져와서 사용하는 행위자체를 막아버리면 어플리케이션이 동작하기 힘들다. 따라서 출처가 다른 리소스(자원)를 사용할 수 있도록 하는 몇가지 예외 조항이 있는데 그중 하나가 바로 `CORS` 정책을 지킨 리소스 요청이다.

서버측에서 HTTP 응답 헤더인  `Access-Control-Allow-Credentials` 에 허용하는 출처를 넣어서 요청을 허용하는지의 여부를 알려주고,  응답을 받은 브라우저는 자신이 보냈던 요청의 `Origin`과 서버 응답의 `Access-Control-Allow-Origin` 값이 일치하는지 확인하고 일치하면 유효한 요청이라고 판단해 받아온 리소스를 사용한다.

## 3.1 CORS의 동작방식

`CORS`의 동작방식은 크게 3가지로 나눌 수 있다.

### 1. 단순요청 (Simple Request)

![simple request](https://github.com/princenim/TIL/assets/59499600/248775ef-f18a-425e-9232-5af3928dd7f8)

요청과 응답을 한번 주고받는 것을 의미한다. 하지만 한번 일어나는 만큼 안정성을 보장할수있도록 조건이 까다롭다.

- HTTP 메소드가 `GET`,`HEAD`, `POST`중 하나여야한다.
- `User Agent`가 자동으로 설정한 헤더를 제외하면 `Accept`, `Accept-Language`, `Content-Language` ,`Content-Type`, `DPR`, `Downlink(en-US)`, `Viewport-Width`, `Width` 만 사용할 수 있다.
- `Content-Type` 헤더에서는 `application/x-www-form-urlencode` , `mulipart/form-data`, `text/plain` 의 값만 설정할 수 있다.

`사전 요청(Preflight Request)` 단계를 거치지 않고 바로 서버에 요청을 전송할 수 있어 효율적인 통신이가능하다. 서버는 응답 헤더에 허용되는 브라우저의 주소를 명시해서 응답한다. 그리고 브라우저는 허용된 출처의 요청에 대해서만 웹 페이지의 결과를 표시한다.

```bash
HTTP/1.1 200 OK
Content-Type: application/json
Access-Control-Allow-Origin: http://example-origin.com //허용하는 URL
```

### 2. 사전 요청(Preflight Request)

![preflight](https://github.com/princenim/TIL/assets/59499600/3c7f63fe-6176-4179-a385-1df2024cd125)

사전 요청은 본 요청을 보내기전에 사전 요청을 보내서 서버가 이에 대해 응답이 가능한지 확인하는 방법이다.안전한 요청임을 확인하면 그때서야 실제요청을 보내므로 총 2번의 요청을 전송한다.

사전 요청은 항상 `OPTIONS 메소드`를 사용한다. 그리고 헤더에는 `Origin`, `Access-Control-Request-Method`, `Access-Control-Request-Headers` 등이 포함된다. 서버는 사전요청을 받아서 실제 요청을 허용할지 여부를 판단하고 이에 대한 응답으로  `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` 등을 포함하는 헤더를 반환한다.

응답하는 헤더는 다음과 같다.

- `Access-Control-Allow-Origin`**:** 요청이 허용되는 출처의 목록을 나타내는 헤더.
- `Access-Control-Allow-Methods`**:** 허용되는 HTTP 메서드의 목록을 나타내는 헤더.
- `Access-Control-Allow-Headers`**:** 허용되는 헤더의 목록을 나타내는 헤더.
- `Access-Control-Allow-Credentials`**:** 인증 정보를 요청에 포함할 수 있는지 여부를 나타내는 헤더.
- `Access-Control-Max-Age`**:** 사전 요청의 결과를 캐싱할 수 있는 시간(초)을 나타내는 헤더.

```bash
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://example-origin.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 86400
```

### 3. 인증 정보를 포함한 요청(Credential Request)

![credential](https://github.com/princenim/TIL/assets/59499600/eb312c36-70a8-4e49-a83d-1b3803a76018)

이 요청은 인증정보(credential)을 함께 보내는 요청을 말한다. 인증정보에는 `쿠키`와 `HTTP 인증 헤더`를 의미한다.

이 요청을 사용하려면  보안상의 이유로 민감한 정보를 담은 요청을 보낼때 추가적인 규칙을 따라야한다. 반드시 서버측에서 `Access-Control-Allow-Credentials` 를 `true`로 설정해야한다. `Access-Control-Allow-Origin` 에 `*` 를 사용하지 않고 url 를 정확히 설정해야한다. 이렇게 하면 브라우저는 실제 요청을 보낼때 자격 증명을 보낼 수 있다.

```bash
#응답
HTTP/1.1 200 OK
Content-Type: application/json
Access-Control-Allow-Origin: http://example-origin.com
Access-Control-Allow-Credentials: true
```

HTTP 인증 헤더는  `Authorization` 헤더를 사용하는 경우를 말한다. `Authorization` 헤더는 일반적으로 HTTP 기반의 인증 방식(Basic, Bearer 등)에 사용된다.

```bash
#요청 
GET /example HTTP/1.1
Host: example.com
Authorization: Basic base64encoded(username:password)
```

## 3.1 CORS 에러

`CORS 에러`는 브라우저에서 발생하는 오류로 `CORS` 정책 위반시에 나타난다. 이 에러는 프론트와 백엔드로 나누어 개발할 경우 흔히 나타난다.
참고로 `localhost`는 현재 사용중인 컴퓨터를 가리킨다. 특별한 ip로 `127.0.0.1`과 동일한 역할을 한다. 예를 들어 브라우저에서 `http://localhost` 를 웹 브라우저에서 열면 현재 컴퓨터에 설치된 웹 서버에 접속하게 된다. 따라서 내 로컬에서 개발할때는 CORS 에러가 날 수 없다. `localhost` 또는 `127.0.0.1`가 동일 출처로 간주되기때문이다. 

Spring에서  `CORS` 에러를 해결할 수 있는 방법은 다음과 같다.

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