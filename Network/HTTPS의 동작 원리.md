# **📌**  HTTPS의 동작 원리

# 1. HTTP (Hyper Text Transfer Protocol)

먼저 `HTTPS`를 알기전에는 `HTTP`를 알아야한다.

`HTTP`는 클라이언트와 서버간에 데이터를 주고받는 프로토콜로 텍스트, 이미지, 영상 JSON 등 모든 형태의 데이터를 전송할 수있다. HTTP 는 어플리케이션 레벨의 프로토콜로 TCP/IP 위에서 작동한다. HTTP는 상태를 가지고 있지 않은 `stateless` 프로토콜 기본적으로 `request`(요청), `response`(응답) 구조로 되어있다.

하지만 `HTTP`는 암호화 되지 않은 평문 데이터를 전송하는 프로토콜이기때문에 비밀번호나 주민번호를 주고 받으면 제 3자가 정보를 조회할 수 있다는 문제점이 있었다. 이러한 문제를 해결하기 위해 `HTTPS`가 등장했다.

# 2. HTTPS (Hyper Text Transfer Protocol Secure)

<img width="1024" alt="http" src="https://github.com/princenim/TIL/assets/59499600/775868d0-aa4f-4af0-8af1-63ef6518908a">

HTTPS는 HTTP에 데이터 암호화를 추가했다. 보안 문제를 HTTP에 SSL(보안 소켓 계층) 인증서를 이용해 해결했다. 그리고 이 암호화 과정에서 **공개키(비대칭)암호화, 비밀키(대칭)암호화** 방식을 사용한다.

- **대칭키 암호화** : 클라이언트와 서버가 동일한 키를 사용해 암호화/복호화를 진행한다. 키가 노출되었을때 매우 위험하지만 연산 속도가 빠르다.
- **비대칭키 암호화** : 1개의 쌍으로 구성된 공개키와 개인키를 각각 암호화/복호화 하는데 사용한다. 키가 노출되어도 비교적 안전하지만 속도가 느리다.

또한 서버의 신원을 보증한다. HTTPS는 인증기관을 통해 인증된 인증서를 사용한다. 이는 사용자가 접속한 웹 사이트가 실제로 사용자가 기대한 웹사이트인지 보장한다. 예를 들어 로컬 DNS 파일에서 127.0.0.1을 naver.com이라고 매핑하면 실제로 naver.com을 로컬에서 요청하면 내 컴퓨터로 이동한다. 하지만 이는 인증서 검증을 거치지 않았기 때문에 브라우저는 서버의 신원이 올바르지 않다고 판단하게 된다.

## 2.1 HTTPS 동작과정

HTTPS는 **대칭키 암호화**와 **비대칭키 암호화**를 모두 사용해 빠른 연산속도와 안정성을 모두 얻고있다. 브라우저(클라이언트)와 서버가 같은 세션키를 갖고 빠른 연산을 높여 데이터를 교환하기 위해 대칭키를 사용하며,

처음에 연결을 성립해 세션키를 공유하는 과정에서 비대칭키가 사용된다.

![https](https://github.com/princenim/TIL/assets/59499600/8551d85d-2bf2-4595-ba22-a60ed865e01e)

1. 클라이언트(브라우저)는 서버에게 연결 요청을 한다.
2. 서버는 클라이언트에게 인증서를 전송한다. 이 인증서에는 서버의 정보와 공개키가 포함되어있다.
3. 브라우저는 공개키를 가지고 인증서의 유효성을 검증한다. 이때 브라우저는 주요 인증기관 리스트의 공개키를 가지고 있는데 리스트에서 공개키를 얻을 수 있다. 그리고 세션키를 발급한다.
4. 검증이 완료되면 브라우저는 세션키를 보관하고 서버로부터 받은 공개키로 세션키를 암호화해서 서버로 전송한다.
5. 서버는 암호화된 세션키를 개인키로 복호화해서 세션키를 얻는다.
6. 이제 클라이언트와 서버는 동일한 세션키를 가지고 공유하므로 데이터를 전달할때 세션키를 가지고 암호화/복호화한다.(대칭키 암호화)

이전에 서버는 HTTPS를 위해서 다음과 같은 과정을 거친다.

1. 서버는 SSL 프로토콜을 위해서 공개키와 개인키를 생성한다. (비대칭키 암호화)
2. 서버는 SSL 인증서를 발급받기 위해 CA(인증기관)에게 서버의 공개키 및 각종 서버의 정보를 전달한다.
3. CA는 서버로 부터 받은 정보를 가지고 SSL인증서를 발급한다. 이 인증서에 서버의 공개키와 서버 정보가 포함된다.
4.  CA는 자신의 개인키로 인증서에 서명한다. 그리고 암호화한 인증서를 서버에게 전달한다.

# 3. HTTP와 HTTPS의 차이

- 둘다 TCP/IP위에서 작동하는 애플리케이션 레벨 프로토콜이다.
- HTTP는 암호화되어있는 않은 평문 데이터를 전송하기때문에 보안상 취약점이 있으며, 이에따라 SSL이라는 보안 계층을 전송계층위에 올려서, SSL위에  HTTP를 얹어서 통신을 하게 되었고 이게 HTTPS이다.
- HTTP는 별도의 암호화 과정이 없기때문에 속도가 빠르고, HTTPS는 암호화과정이 필요가기떄문에 HTTP에 비해 느리다. 개인정보와 같은 민감한 데이터를 주고받아야한다면 HTTPS, 노출이 되어도 상관없는 단순한 정보조회라면 HTTP를 이용하면 된다.
- HTTP는 80포트 , HTTPS는 443포트를 사용한다.