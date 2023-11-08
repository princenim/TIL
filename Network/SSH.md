# SSH (Secure Shell Protocol)

- 기존에 유닉스 시스템 셸에 원격 접속하기 위해 사용하던 **텔넷**(telnet)은 암호화가 이루어지지않아 계정정보가 탈취될 위험이 높았는데 여기에 암호화 기능을 추가해 19995년에 나온 프로토콜이다.
- port는 22
- 맥과 리눅스는 기본적으로 설치되어 있으며, 윈도우는 PuTTY 나 Xshell을 통해 사용가능하다.

## 인증방식

- 원격 호스트에 접속하기 위해 사용되는 인터넷 프로토콜이다.
- ssh는 일반적으로 비밀번호 입력을 통한 접속을 하지 않고, key 를 통해 통신을한다.  공개키 , 비공개 키로 이루어지며 비공개키는 로컬에 위치하고, 공개키는 연격 server에 위치한다. 따라서 SSH 접속을 시도하면 로컬의 비공개키와 원격서버의 비공개키를 비교해서 일치하는지 인증과정을 거치고 접속을 하게 된다.



## 사용 예시

1. 데이터 전송 : 깃허브에  소스코드를 깃헙에 푸쉬할때 SSH 활용
2. 원격 제어 : AWS, NCP 같은 클라우드 서비스의 인스턴스 서버에 접속해 명령을 내리기 위해 SSH 를 통한 접속.
    - 인스턴스를 생성하면 pem 파일을 받게 되는데 이는 SSH 보안방식이 적용된 파일로 이 파일을 서용해 원격에 접속할 수 있다. 이 파일을 비공개키라고 생각하면 된다. 즉 공개키가 AWS에 비공개키인 pem파일이 로컬에 존재하는 것이다.