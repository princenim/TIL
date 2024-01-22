# **📌** volatile 키워드

# 1. volatile

`volatile` 키워드는 자바에서 멀티쓰레딩 환경에서 변수의 값을 읽거나 쓸때 해당 변수의 값을 쓰는 쓰레드에서 **캐시를 사용하지 않고 항상 메인 메모리에서 읽거나 쓰도록 보장한다.** 이를 통해 변수에 대한 변경이 다른 쓰레드에 즉시 반영되도록한다.

```java
volatile boolean suspended = false;
```

![volatile](https://github.com/princenim/TIL/assets/59499600/8b05a6ba-3d6d-44c6-81ce-f36ea9c36bba)


멀티 코어 프로세서에서는 코어가 2개이상이며 각각의 코어는 모두 메모리에서 읽어온 값을 캐시에 저장하고, 값을 읽어서 작업한다. 다시 같은 값을 읽어올때는 캐시에 있는지 확인하고 없을 때만 메모리에서 가져온다. 따라서 메모리에 저장된 변수의 값이 변경되었는데도 캐시에 저장된 값이 갱신되지 않아서 변수의 값이 다른 경우가 발생한다. 이때 `volatile` 를 사용하면 코어가 변수의 값을 읽어올때 캐시가 아닌 메모리에서 읽어오기 때문에 캐시와 메모리 간의 불일치가 해결된다.

# 2. volatile 변수의 특징

## 2.1 가시성(Visibility) 보장

`volatile` 키워드는 변수의 가시성을 보장해준다. 가시성은 말 그대로 눈에 쉽게 보이는 정도를 뜻하는데 컴퓨터 과학에서는 멀티스레드 환경에서 각각의 쓰레드가 공유자원에 대해서 모두 같은 상태를 바라보고 있는 것을 말한다. 즉, **하나의 쓰레드에서 해당 변수를 수정한 내용이 다른 쓰레드에서 즉시 반영되도록 보장한다.**

코드를 예를 들어보면

```java
public class SharedResource {
    private int sharedValue = 0;

    public void writeValue() {
        // 변수에 값을 쓰기
        sharedValue = 42; // 스레드 A에서 쓰기 작업 수행
    }

    public int readValue() {
        // 변수에서 값을 읽기
        return sharedValue; // 스레드 B에서 읽기 작업 수행
    }
}
```

`volatile`의 키워드를 사용하지 않는 코드이다. `writeValue()` 메소드에서 `sharedValue` 에 값을 쓰는 `쓰레드 A`가 존재할때 `쓰레드 A`가 `sharedValue`을 쓰는 순간에는 이 값을 CPU의 캐시에 저장할 수 있다. 따라서 메인 메모리와의 일관성이 보장되지 않는다.

`readValue()` 메소드에서 `sharedValue`를 읽는 `쓰레드 B`가 존재할때, `쓰레드 B`는 `sharedValue` 를 읽을 때 CPU 캐시를 확인할 가능성이 높다. 따라서 이미 저장된 CPU 캐시안에 저장된 값이 메인메모리의 최신값과 일치하지 않을 수 있다.

밑의 코드는 `volatile` 키워드를 사용한 코드이다.

```java
public class SharedResource {
    private volatile int sharedValue = 0;

    public void writeValue() {
        // 변수에 값을 쓰기
        sharedValue = 42; // 스레드 A에서 쓰기 작업 수행
    }

    public int readValue() {
        // 변수에서 값을 읽기
        return sharedValue; // 스레드 B에서 읽기 작업 수행
    }
}
```

`쓰레드 A`에서 쓰기 작업을 하면 `writeValue()` 메소드에서 `sharedValue`를 쓰는 시점에서 해당 값을 CPU캐시에서 저장하지 않고 바로 메인 메모리에 즉시 반영한다.

`sharedValue` 를 읽는 `쓰레드 B`가 존재할때 `sharedValue` 를 읽을 때 메인 메모리에서 직접 값을 읽어온다.다시말해  다른 쓰레드에서 변경한 값이 메인 메모리에 즉시 반영되므로 캐시 일관성 문제를 해결할 수 있다.

## 2.2 원자성(Atomicity)

**원자성이란 작업을 더 이상 나눌 수 없다는 의미이다.** `volatile`을 사용하면 해당 변수에 대한 읽기나 쓰기가 원자화된다.

더 자세히 설명하면 JVM은 데이터를 4byte 단위로 처리한다. 따라서 int와 int 보다 작은 타입들은 한번에 읽거나 쓰는 것이 가능하다. 즉, 단 하나의 명령어로 읽거나 쓰기가 가능하다는 의미이다. 하나의 명령어는 더 이상 나눌 수 없는 최소의 작업단위이므로, 작업 중간에 다른 쓰레드가 끼어들 틈이 없다.

그러나 `long`과 `double` 타입의 변수는 8byte이므로 하나의 명령어로 값을 읽거나 쓸수 없기 때문에 변수의 값을 읽는 과정에서 다른 쓰레드가 끼어들 여지가 있다. 변수 할당 및 연산 시 일어나는 작업이 여러번에 걸쳐 메모리 작업이 일어난다는 의미이다. 예를 들어 `long`형이 64비트인데 작업단위가 32비트단위이므로 첫 32비트 할당, 다음 32비트를 할당한다. 하지만 첫 32비트를 할당했을 때 다른 쓰레드가 값을 읽어가면 정상적이지 않은 상황이 발생할 수 있다.  이때 `volatile`을 키워드를 붙여 원자화함으로써  작업을 더 나눌 수 없게 할 수 있다.

```java
volatile long val1; //long 타입의 변수를 원자화 
volatile double val2; //double 타입의 변수를 원자화 
```

`synchronized` 블록도 일종의 원자화라고 할 수 있다. `synchronized` 블럭은 여러 문장을 원자화함으로써 쓰레드의 동기화를 구현한 것이다.

여기서 중요한 점은 `volatile` 은 변수의 읽기나 쓰기를 원자화할 뿐 동기화 하는것은 아니다.