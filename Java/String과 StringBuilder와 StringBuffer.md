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

두 클래스는 모두 같은 메소드를 가지고 있는데 한가지 차이점이 있다 `**StringBuffer`는 멀티 쓰레드에 안전하고 ,  `StringBuilder`은 멀티쓰레드에 안전하지 않다.**

## 2.1 StringBuffer

`StringBuffer`은 멀티쓰레드에 안전하다. thread-safe하다.  각 메서드별로 `synchronized` 키워드가 존재하여 동기화가 보장되어있다. 즉, 이 말은 쓰레드가 진행중인 작업을 다른 쓰레드가 간섭하지 못한다는 의미이다.

## 2.2 StringBuilder

`StringBuilder`는 `StringBuffer`와는 다르게 멀티쓰레드에 안전하지 않다. 하지만 단일쓰레드 환경에서는 속도가 더 빠르게 때문에 동기화를 고려하지 않는 환경에서 사용하는 것이 좋다.

# 3. 정리

![string](https://github.com/princenim/TIL/assets/59499600/c8b0bcea-6ba6-4950-9f81-7e93d24181f6)


즉 위의 내용을 바탕으로 정리하면 다음과 같을때 사용하는 것이 좋다.

- `String` : 문자열 연산이 적고, 멀티쓰레드환경일 경우
- `StringBuffer` : 문자열 연산이 많고 멀티쓰레드 환경일 경우
- `StringBuilder` : 문자열 연산이 많고 단일쓰레드이거나 동기화를 고려하지 않아도 되는경우