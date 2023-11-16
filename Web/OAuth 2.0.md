# OAuth 2.0

## OAuth 2.0 개념

Open Authorization 2.0의 약자로 인증을 위한 프로토콜이다.

제 3자 인증방식으로 구글 **, 카카오, 네이버 ,페이스북, 깃허브** 등에서 제공하는 간편 로그인 기능을 말한다.

기본적으로 사용자는 서버를 신뢰하지 않고, 자신의 민감한 정보를 작성하는 것을 꺼린다. 서버 측 또한 마찬가지로 사용자의 민감한 정보를 관리하는데 리소스가 필요하다.  따라서 OAuth를 사용해 신뢰할 수 있는 서버에  정보를 맡겨놓고 접근 할 수 있는 권한을 주는 것을 말한다.

## OAuth 2.0 주체

`Resource Owner` :  사용자

`Resource Server` & `Authorization Server`: OAuth를 통해 인증, 인가를 제공해주는 서버 를 말한다. 사용자의 자원(이름, 이메일)을 제공해준다. e.g. 카카오, 네이버 , 구글

`Client Application` : 사용자가 사용하는 서비스 어플리케이션을 말한다.

## OAuth 2.0 의 종류

4가지가 존재하며  ***Authorization Code Grant의*** 방식을 가장 많이 사용한다.

- ***Authorization Code Grant (인증 코드 허가)***
- Implicit Grant
- Resource Owner Password Credentials Grant
- Client Credential Grant

## Authorization Code Grant 방식

![images-oauth_auth_code](https://github.com/princenim/TIL/assets/59499600/7ce31ae1-380d-4a4c-bf93-1965f5c7bfe3)


1. `Client Application`이 `Authorization Server` 로 인증 요청을 보낸다.
2. `Authorization Server` 는 유효한 인증인지 확인 후,  `Authorization code`를 발급한다.
3. `Client Application` 은 이 `Authorization code` 를 가지고 다시 `Authorization Server` 에게  `access token` 을 요청한다.
4. `Authorization code` 는 확인 후 `access token` 을 발급한다.
5. `Client Application` 는 이 `access token` 을 담아서 , `Authorization Server` 에게 사용자의 정보를 요청한다.
6. `Authorization Server` 는 `access token` 확인 후 , 사용자의 정보를 전달한다 .
7. `Client Application` 은 이 사용자의 정보를 가지고 서비스 어플리케이션에서 사용할 Access Token & Refresh Token 을 발급한다.

따라서 OAuth2.0 를 통해 SNS 로그인 기능을 구현했어도 실제 서비스애플리케이션의 인증방법은 별도로 필요하다. 즉, 세션,쿠키,토큰을 사용해 별도로 인증 작업을 해줘야한다.  (7번을 말한다.)

## OAuth 와 JWT의 차이

JWT는 토큰의 한 종류이며, OAuth는 토큰을 발급하고 인증하는 오픈 스탠다드 프로포콜을 말한다. 토큰을 요청할 때 사용할 수 있어야하는 요청 및 응답순서와 형식이 존재한다. 따라서 OAuth는 JWT와 달리 토큰 자체에 큰 의미를 가지고 있지 않고 권한을 확인하고 , 서비스 제공자에게 데이터를 요청하기 위해 사용한다.