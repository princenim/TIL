# **📌** I/O의 stream과 buffer

# 1. I/O

`IO`는 `Input(입력)`, `Output(출력)`의 약자로 컴퓨터 내부 또는 외부의 장치와 프로그램간의 데이터를 주고받는 것을 말한다. 예를 들면 키보드로 부터 데이터를 입력받는다거나 `System.out.println()`을 이용해 화면에 출력하는 것이 가장 대표적인 입출력의 예이다.

# 2. 스트림 (stream)

자바에서 입출력을 수행하려면 즉 어느 한쪽에서 다른 쪽으로 데이터를 전달하려면 두 대상을 연결하고 데이터를 전송할 수 있는 무언가가 필요한데 이것을 `스트림(stream)`이라고 정의했다. 즉, **스트림(stream)이란 데이터를 운반하는데 사용되는 연결통로이다.**

데이터를 보내는 쪽과 받는쪽이 둘다 스트림에 연결되어있을때 데이터를 주는 쪽에 데이터를 보내는 동안 받는 쪽은 데이터를 받아서 처리하는 작업을 진행하며 이 작업동안은 다른 작업을 진행할 수없다. 따라서 이는 blocking이다.

![stream](https://github.com/princenim/TIL/assets/59499600/fa2ee54f-3cec-4433-a32e-e93a417cee28)

물이 한쪽으로만 흐르는 것과 같이 스트림은 단방향 통신만 가능하기 때문에 하나의 스트림으로 입력과 출력을 동시에 처리할 수 없다. 그래서 입력과 출력을 동시에 수행하려면 입력을 위한 `입력스트림(input stream)`, 출력을 위한 `출력스트림(output stream)` 모두 2개의 스트림이 필요하다. 스트림은 먼저 보낸 데이터를 먼저 받게 되어 있으며 중간에 건너뀜 없이 연속적으로 데이터를 주고받는다. 큐와 같이 FIFO 구조로 되어있다고 생각하면된다.

스트림은 바이트 단위와 데이터를 전송하며 입출력 대상에 따라 다음과 같은 입출력 리스트가 있다. 모두 추상 클래스인 `InputStream`과 `OutputStream`을 상속받은 클래스이다.

| 입력 스트림 | 출력 스트립 | 입출력 대상의 종류  |
| --- | --- | --- |
| FileInputStream | FileOutputStream | 파일 |
| ByteArrayInputStream | ByteArrayOutputStream | 메모리(byte배열) |
| PipedInputStream | PipedOutputStream | 프로세스(프로세스간의 통신) |
| AudioInputStream | AudioOutputStream | 오디오장치 |

## 2.1 보조스트림

스트림 외에도 스트림의 기능을 보완하기 위한 보조스트림이 있다. 실제 데이터를 주고받는 스트림은 아니지만 스트림의 기능을 향상시키거나 새로운 기능을 추가할 수 있다.

예를들어 예를 들어, `BufferedReader`와 `BufferedWriter`는 기본 스트림(`FileReader` 및 `FileWriter`)을 감싸고 있어서 버퍼링 기능을 제공한다. 버퍼링은 작은 블록 단위로 데이터를 읽거나 쓰는 것보다 큰 블록 단위로 읽거나 쓰는 것이 효율적이기 때문에 성능을 향상시킬 수 있다.

```java
//파일에 쓰기 - BufferedWriter
public class BufferedWriterExample {

    public static void main(String[] args) {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
            writer.write("Hello, World!");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

이 코드에서 `BufferedWriter`는 `FileWriter`를 기반으로 파일에 쓰고 있다. `BufferedWriter`는 기본 스트림인 `FileWriter`를 감싸고 있어서 버퍼링 기능을 제공한다. 쓰기 작업이 일정 크기 이상이 되거나 작업이 끝나면 내부적으로 **flush**가 호출되어 데이터가 실제 파일에 쓰여진다. 즉 보조스트림을 사용하면 입출력 성능을 향상시킬 수 있다.

# 3. 버퍼(buffer)

버퍼는 데이터를 임시로 저장하는 메모리 공간을 말한다. 입출력작업에서 데이터를 한번에 처리하기 위해 사용된다. 입출력 성능을 향상시키기 위해 도입되었으며 작은 크기의 데이터를 여러 번 입출력하는 것보다 큰 블록 단위로 입출력하는 것이 효율적이다. 예를 들어 마트에서 계산할때 진열대에서 물건을 고를때마다 계산대에 가서 하나씩 옮기는 것보다 카트에 모아뒀다가 한번에 계산대로 가져가는 것과 같은 원리이다.

버퍼를 활용한 `I/O`의 가장 대표적인 클래스는 `BufferedReader`, `BufferedWriter`, `BufferedInputStream`, `BufferedOutputStream` 가 있다.

```java
public class BufferedReaderExample {
    public static void main(String[] args) {
        try (BufferedReader reader = new BufferedReader(new FileReader("example.txt"))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

위의 코드는 `BufferedReader`를 사용해 파일에서 텍스트 파일를 읽어오는 예시이다.

```java
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
bufferedReader.readLine();
```

위는 버퍼를 사용해 콘솔에서 입력을 받는 예시이다.

`System.in` 은 콘솔로부터 입력을 받는 `InputStream`, 그리고 `InputStreamReader`는 받은 바이트 스티림을 문자스트림으로 변환한다. 그리고 `BufferedReader`는 문자열을 버퍼를 저장하여 성능을 향상시킨다. 그리고 `readLine` 은 버퍼에 저장된 데이터를 한줄씩 읽는다.  그리고 `readLine` 는 사용자가 입력을 완료할때까지 대기하므로 이 부분에서 `blocking` 상태가 된다.

## 3.1 버퍼를 사용하는 이유

1. **성능 향상** : 작은 데이터 단위를 입출력할때마다 `I/O`작업을 수행하면 성능이 저하된다. 하지만 버퍼를 사용하면 큰 블록 단위로 데이터를 처리하므로 `I/O` 작업 횟수를 줄여 성능을 향상시킬 수 있다.
2. **자동 Flush** : 버퍼는 일정 크기 이상의 데이터가 모이거나 작업이 끝날 때 자동으로 `flush`를 수행한다. `flush`는 버퍼에 모인 데이터를 출력 장치로 보내는 동작으로 효율적인 입출력을 가능하게 한다.