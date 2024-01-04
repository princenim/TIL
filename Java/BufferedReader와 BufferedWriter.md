# 1. BufferedReader  & BufferedWriter

`BufferedReader`와 `BufferedWriter` 는 Buffer에 있는 IO 클래스이다. 이름처럼 **버퍼를 통해 읽고 쓰는 함수로 입출력 데이터가 바로 전달되지 않고 중간에 버퍼링이 된후 전달된다.** 그렇기 때문에 매우 빠르다. 예를들면 마트에서 계산할때 물건마다 진열대에서 계산대로 가서 하나씩 옮기는 것보다 카트에 모아뒀다가 한번에 계산대로 가져가는것과 같은 원리이다.

- **버퍼(buffer)** : 데이터를 한 곳에서 다른 한 곳으로 전송하기 전에 일시적으로 데이터를 보관하는 임시 메모리 영역

## 1.1 BufferedReader

`Scanner`처럼 문자열을 입력받는 클래스이다. 단, 데이터가 바로 전달되지 않고 중간에 버퍼링 된후 전달된다.
만약 정수 하나를 입력받는다고 하면 이렇게 받을 수 있다.

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
int n = Integer.parseInt(br.readLine()); //리턴타입이 string이기 때문에 int로 변환
```

- `system.in` : 사용자로부터 입력받은 값을 InputStream 형태로 얻는다. 즉 리턴이 바이트 스트림인InputStream
- `InputStreamReader` : InputStream은 바이트 단위로 값을 읽어오는데 이를 문자 단위로 데이터를 변환시켜준다.
    - e.f. 입력 : abc , 출력 : abc
- `BufferedReader(Reader rd)` : rd에 연결되는 문자입력 버퍼스트림생성. InputStreamReader는 문자열이 아닌 문자를 처리하는데 만약 문자열을 처리하고 싶다면 배열을 선언해야한다는 단점이 존재한다.  이를 개선해 버퍼를 통해 입력받은 문자를 쌓아둔 뒤 한번에 문자열처럼 보내버린다.
- `readLine` : 입력 메소드로 한줄을 전부 읽어오며 리턴타입이 String이다.
    - 비슷한 예로 Scanner 메소드 중 `nextline()`도 개행문자를 포함한 한줄을 전부 읽는다.

즉 정리하면 System.in = InputStream -> InputStreamReader -> BufferedReader 은 byte 타입으로 읽어들여 char타입으로 처리 그리고 String 으로 저장한다.

또한 BufferedReader는 예외처리 **IOException**를 꼭 해줘야한다.

## 1.2 BufferedWriter

`System.out.println()` 처럼 문자열을 출력하는 클래스이다.

```java
BufferedWriter bw = new BufferedWriter(new OutputStreamWriter((System.out))); //선언
bw.write(a + b + "\n"); //출력 버퍼 스트림에 문자열을 모두 보낸다. 
bw.flush(); //모두 출력 즉 버퍼를 비우는 역할
bw.close();; //닫음.
```

- `System.out` : system.in과 반대로 System.out으로 출력 작업 수행
- `OutputStreamWriter` : 주어진 출력 바이트 스트림 out에 대해 기본 인코딩을 사용하는 객체를 생성한다.
- `BufferedWriter(Writer wt)` :wt에 연결되는 문자 출력 버퍼스트림생성. 즉 버퍼에 모아놓은 문자열을 출력가능하게 해준다.
- `write()`: 출력 버퍼 스트림에 모아둔 문자열을 모두 보낸다. 또한 개행이 안 되기 떄문에 "\n"추가해야한다.
- `flush()` : 버퍼에 잔류하는 문자열을 모두 출력한다. 버퍼를 비우는 역할
- `close()`: 사용한 시스템 자원을 반납하고 출력 스트림을 닫는다. 출력스트림을 더 이상 사용하지 않을 경우에 호출한다.

## 1.3 BufferedReader가 Scanner보다 빠른 이유

버퍼를 사용하면 더 느릴것 같지만 더 빠르다. 그 이유는 하드디스크는 원래 속도가 느린데 하드디스크 뿐만 아니라 키보드나 모니터와 같은 외부 장치와의 데이터 입출력은 시간이 많이 걸린다. 따라서 키보드를 누를때마다 문자의 정보를 바로 입력 및 출력시키는 것보다 중간에 버퍼를 통해 한번에 이동시키는 것이 빠르다.

그렇다면 BufferReader가 항상 좋을까? 아니다 BufferReader에도 단점이 존재하는데

1. BufferedReader는 모든 입력 데이터를 String으로 인식한다. 따라서 입력받은 문자열을 분석하고 파싱해야한다.
2. BufferReader는 무조건 try-catch와 throw를 통해 예외 처리를 해줘야한다.

따라서 많은 양을 출력하고 속도가 중요할때 `BufferedReader`,`BufferedWriter`을 사용하자