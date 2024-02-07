# **📌** String과 StringBuilder와 StringBuffer

`String`과 `StringBuider`와 `StringBeffer`은 모두 자바의 문자열 클래스이다. 하지만 굳이 여러가지를 만들어 놓은 이유가 무엇일까?

# 1. String

`String`과 다른 두 `StringBuffer`,`StringBilder`의 큰 차이점은 바로 `String`은 `불변객체라는` 것이다.  불변객체란 `Immuatble` 이라는 의미로 말 그대로 변경이 불가능하다는 의미로 객체가 생성된 후 내부의 상태가 변하지 않고 계속 유지되는 객체를 말한다. 따라서 + 연산이나 concat 메소드를 사용할때 메모리상에 기존에 생성된 `String`객체 문자열에 다른 문자열을 붙여도 기존 문자열에 새로운 문자열이 붙는데가 아니라 새로운 String 객체를 만든후 새 `String` 객체에 다른 문자열을 연결하고 그 객체를 참조하도록 한다.

참고로 `String 리터럴`과 `new`를 이용한 스트링 객체 생성 방법 존재한다. 둘다 `heap`영역에 저장되나 리터럴은 `heap` 영역의 `String constant pool`영역에 저장된다.

따라서 내부적으로 계속 새로운 객체를 만들기때문에 문자열 연산이 많은 경우에는 성능이 좋지않다. 이런 점을 해결하기 위해 등장한 것이 `StringBuffuer`과 `StringBuilder`이다.



## 1.2 String이 불변객체인이유

하지만 왜 자바는 String을 불변객체로 만들었을까?

1. **`보안 및 안전`** : String은 민감한 정보를 저장하기 위해 많이 사용된다. 네트워크 연결시의 Host,Port의 정보나 파일 및 디렉토리 경로 정보등을 String으로 저장한다. 따라서 만약에 String이 불변하지 않고 mutable할때 메소드의 호출자에 의해 값이 언제든 바뀔 수 있으므로 보안상의 취약점을 갖게된다.
2. **`동기화(synchronization)`**  : 불변객체는 값이 바뀌지 않기때문에 멀티쓰레드 환경에서 `Thread-safe`하다.  쓰레드가 값을 변경하면 동일한 `String`을 수정하지 않고 `String constant pool`에 새 문자열이 생기기 때문이다. 따라서 멀티 쓰레드 환경에서 안정을 가질 수 있다.
3. **`성능`** : String은 리터럴로 객체를 만들었을때 값을 String constant pool에 저장한다. 그리고 같은 문자열이 있는지 먼저 String constant pool에서 확인을 하고 없으면 새로운 String 객체를 String constant pool에 만들고 있으면 기존의 String의 주소값을 반환한다. 이렇게 문자열 리터럴을 캐싱하고 재사용하면 힙 공간을 많이 절약할 수 있다.

   하지만 이때 String이 불변객체가 아니면서 String constant pool을 사용한다면 기존의 문자열 값을 수정해서 문자열의 값이 달라진다면 같은 객체를 바라보는 다른 참조변수들도 바뀐 값을 바라보게 된다. 이는 말도 안되는 것이므로 String은 불변객체일수 밖에 없다.


# 2. StringBuffer와 StringBuilder

두 클래스는 String 클래스와 다르게 `가변(mutable)` 객체이다. 객체의 공간이 부족할때 버퍼의 크기를 유연하게 늘려 저장한다는 의미이다. 값이 변경될때 `String` 객체와 같이 새로운 객체를 만들지 않고 기존 객체내에서 문자열의 크기를 변경하기 때문에 문자열의 추가, 수정, 삭제할때 사용할때 이점이 있다.

두 클래스는 모두 같은 메소드를 가지고 있는데 한가지 차이점이 있다 `StringBuffer`는 멀티 쓰레드에 안전하고 ,  `StringBuilder`은 멀티쓰레드에 안전하지 않다.

## 2.1 StringBuffer

`StringBuffer`은 멀티쓰레드에 안전하다. thread-safe하다.  각 메서드별로 `synchronized` 키워드가 존재하여 동기화가 보장되어있다. 즉, 이 말은 쓰레드가 진행중인 작업을 다른 쓰레드가 간섭하지 못한다는 의미이다.

## 2.2 StringBuilder

`StringBuilder`는 `StringBuffer`와는 다르게 멀티쓰레드에 안전하지 않다. 하지만 단일쓰레드 환경에서는 속도가 더 빠르게 때문에 동기화를 고려하지 않는 환경에서 사용하는 것이 좋다.
그리고 `StringBuilder`의 객체는 `String` 처럼 `String constant pool`을 사용하지 않고 `heap` 영역에 저장된다.

그렇다면 `StringBuilder`은 연산할때 어떻게 동적으로 크기를 늘리고 `String`보다 빠를까?

### 동작 원리

**`StringBuilder`은 내부적으로 `ArrayList`와 같은 원리로 크기가 동적으로 늘어난다.**

```java
StringBuilder sb = new StringBuilder("Hello"); 
System.out.println(sb.capacity()); //21
```

기본적으로 `StringBuilder` 객체 생성 시 내부적으로 `default capacity`가  `16 bytes`의 버퍼 크기를 가진다. 따라서 `16 + 5`해서 `21`의 용량을 가진다.

```java
sb = sb.append("World"); 
System.out.println(sb.capacity()); //21
```

만약 해당 `StringBuilder`에 크기가 `5`인 문자열을 추가해도 용량은 그대로 `21`이다. 이는 `StringBuilder`이 기존 남은 용량(= `16`)을 사용했다는 의미이다.

그렇다면 만약에 `16` 이상의 문자열을 더하면 어떻게 될까?

```java

sb = sb.append("WorldWorldWorldhh"); //길이가 17인 문자열을 더함
System.out.println(sb.capacity()); //44
```

이때는 남은 용량인 16을 넘어가 `(기존 용량 크기 + 1) * 2` 즉 `(21 +1) * 2 = 44`의 용량을 가진 문자열이된다.

즉 `StringBuilder`은 `ArrayList`처럼 갖고있는 버퍼 용량이 찼을 때 새로운 용량의 크기를 늘려 새로운 공간에 할당하는것을 알 수 있다.

다음은 `JDK16`의 `StringBuilder` 코드이다.

```java
@IntrinsicCandidate
    public StringBuilder() {
        super(16); //default caapcity
  }

AbstractStringBuilder(int capacity) {
        if (COMPACT_STRINGS) {
            value = new byte[capacity];
            coder = LATIN1;
        } else {
            value = StringUTF16.newBytesFor(capacity);
            coder = UTF16;
        }
    }
```

`StringBuilder` 생성 시 용량이 `16`인 `byte 배열`이 내부적으로 생성됨을 알 수 있다.

```java
public AbstractStringBuilder append(String str) {
        if (str == null) {
            return appendNull();
        }
        int len = str.length();
        ensureCapacityInternal(count + len); 
        putStringAt(count, str);
        count += len;
        return this;
    }
```

```java
/**
     * For positive values of {@code minimumCapacity}, this method
     * behaves like {@code ensureCapacity}, however it is never
     * synchronized.
     * If {@code minimumCapacity} is non positive due to numeric
     * overflow, this method throws {@code OutOfMemoryError}.
     */
    private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        int oldCapacity = value.length >> coder;
        if (minimumCapacity - oldCapacity > 0) {
            value = Arrays.copyOf(value, //value는 byte[], 새로만든 byte[]에 복사후 리턴
                    newCapacity(minimumCapacity) << coder);
        }
    }
```

그리고 `append()`메소드 호출 했을 때 `minimumCapacity- oldCapacity` 이 0보다 클때 즉, `minimumCapacity (count + 새로운 문자열의 길이)` 가 기존의 `byte[]`의 길이보다 길다면 기존 `byte[]`에 추가하지 못하고 새로운 `byte[]`만들어야하니 `(기존용량 크기 + 1) * 2`배의 용량의 `byte[]`을 만들어 복사한다.

- `Arrays.copyOf(원본배열, 복사할길이)` : 원본 배열을 0부터 원하는 길이만큼 복사한다.
- `count` : 현재 `byte[]`의 사이즈
- `miminumCapacity`: 최소한으로 필요한 용량,
- `oldCapacity` : 기존 용량크기

# 3. 정리

![string](https://github.com/princenim/TIL/assets/59499600/c8b0bcea-6ba6-4950-9f81-7e93d24181f6)


즉 위의 내용을 바탕으로 정리하면 다음과 같을때 사용하는 것이 좋다.

- `String` : 문자열 연산이 적고, 멀티쓰레드환경일 경우
- `StringBuffer` : 문자열 연산이 많고 멀티쓰레드 환경일 경우
- `StringBuilder` : 문자열 연산이 많고 단일쓰레드이거나 동기화를 고려하지 않아도 되는경우