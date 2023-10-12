# hosts 파일

ip 주소와 도메인 주소를 매핑해주는 파일이다.

호스트 이름에 대응하는 ip주소가 저장되어 있어서 DNS(Domain Name System)에서 주소 정보를 제공받지 않아도 서버의 위치를 찾게 해주는 파일을 말한다.

만약에 이렇게 ip주소와 도메인 주소를 hosts파일에 저장하고, 해당 도메인 주소를 호출하면 실제 [www.me.com](http://www.me.com) 주소로 접속되지 않고, 내가 설정한 111.111.111.111로 접속이 된다.

```go
111.111.111.111 www.me.com
```

### 파일 위치

```go
//윈도우의 호스트파일 위치 
C:\Windows\System32\drivers\etc

//리눅스의 호스트파일 위치 
etc/hosts

//맥의 호스트파일 위치 
/private/etc/hosts
```