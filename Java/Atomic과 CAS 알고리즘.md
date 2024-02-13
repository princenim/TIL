# 📌 Atomic과 CAS 알고리즘

# 1. Atomic

멀티 쓰레드 환경에서는 동시성 이슈를 고려해야한다. 이 동시성 이슈를 해결하기 위한 방법중 하나는 동기화이다.  `synchronized` 키워드를 사용해서 동기화를 구현할 수 있다. `synchronized`를 사용하면 해당 블럭 전체를 lock(점유)하기 때문에 다른 Thread는 아무런 작업을 하지 못하고 기다리게된다. 다시말해 이때 블럭에 접근하는 쓰레드들은 `block` 상태가 된다. 즉 쓰레드들이 `blocking` 되는 과정과 다시 `resuming` 되는 과정에서 시스템의 자원을 사용하게되는데. 바로 이 부분에서 성능 저하가 발생한다.

이러한 문제를 해결하기 위해 `non-blocking` 방식을 사용하여 `원자성`을 보장하기 위한 방법이 `Atomic`이다.

`Atomic`은 `원자성`이라는 뜻으로 어떤 작업이 프로그램 안에서 가장 작은 단위라서 더 이상 다른 작업으로 나누어 지지 않는 것을 말한다.

```java

Integer a = 10;
a++;
```

`a`가 라는 값이 있을때, `a` `플러스` `1`을 하려고 하면 `CPU` 입장에서 총 몇번의 작업을 실행할까?  `CPU` 입장에서 총 3번 작업이 실행된다.

1. 값을 읽고 (read)
2. 값을 증가시키고
3. 값을 저장한다. (write)

근데 만약 값을 저장하려고 하는 시점에 다른 쓰레드가 `a` 값에 12이라고 저장을 했다면 어떻게 해야할까? 내가 값을 저장하려고 했던 시점에서는 `a`의 값이 10이었는데 메모리에 있는 값은 `12`가 되었다.

그렇다면 값은 `11`이 되어야할까 `13`이 되어야할까? 이 방식은 원자성을 보장하지 않는다.

따라서 원자성을 보장하는 `Atomic` 을 사용해 동기화를 할 수 있는데 이 `Atomic`의 핵심원리가 바로 `CAS(Compare And Swap)알고리즘`이다.

자바에서는 `java.util.concurrent.atomic` 패키지에서 원자적인 연산을 지원하는 클래스를 제공하고 이 클래스들은 모두 `CAS(Compare And Swap)알고리즘` 로 구현되어있다.

그리고 `Atomic` 방식은 락을 획득하고 반납하는 과정이 없기 때문에 `synchronized`보다  훨씬 성능이 좋다.

# 2. CAS 알고리즘

멀티 코어 환경이나 멀티 쓰레드 환경에서 각 `CPU`는 메인 메모리에서 변수값을 참조하지 않고 각 `CPU`의 캐시 영역에서 값을 참조한다. 이때 메인 메모리와 캐시에 저장된 값이 다른 경우가 있다.  이렇게 하나의 쓰레드에서 수정한 결과가 다른 쓰레드에게 보이지 않는 문제를 `가시성 문제`라고 한다. 이 가시성 문제를 해결하는 알고리즘이 `CAS 알고리즘`이다.

**현재 쓰레드에 저장된 값(캐시)과 메인 메모리에 저장된 값을 비교해 일치하는 경우 새로운 값으로 교체 , 일치하지 않으면 다시 재시도를 한다.** 이렇게 처리하면 CPU 캐시에서 잘못된 값을 참조할수 없어 가시성 문제가 해결된다.

다시말해 값이 다르다는것은 쓰레드 A가 공유변수에 대해서 계산을 하고 메모리에 반영하기 전에 쓰레드 B가 공유변수를 변경해 메모리에 반영한 경우를 말한다. 즉 쓰레드 A의 계산을 메모리에 반영하면 안된다. 그래서 실패를 하고 다시 시도를 하는 것이다.





![ㅊㅁㄴ](https://github.com/princenim/TIL/assets/59499600/75ef1587-b711-4711-a658-0b199c0925cd)

CAS 알고리즘의 동작 원리는 다음과 같다.

1. 인자로 `기존 값(Compared value)`와 `변경할 값(Exchanged value)`를 전달한다.
2. `기존값(Compared value)`이 `현재 메모리가 가지고 있는 값`과 같다면 `변경할 값(Exchanged value)`을 반영해 `true`를 반환한다.
3. 반대로 `기존 값(Compared value)`이 `현재 메모리가 가지고 있는 값(destination)`과 다르다면 값을 반영하지 않고 `false`를 반환한다.


![cas](https://github.com/princenim/TIL/assets/59499600/4a119b34-86d8-4b9d-a2d0-7f98696a510e)

위의 이미지를 예시로 더 자세하게 설명하면

1. `Thread1`과 `Thread2`는 메인 메모리에있는 `counter` 변수를 읽어 CPU 캐시에 저장한다.
2. 각 쓰레드는 `counter` 값을 연산한다.
3. `Thread1`과 `Thread2`는 연산하고 난 counter값을 메인 메모리에 반영하기 전의 `counter`값과 메인 메모리에 저장된 counter값을 비교한다.
   1. 기존 값이 메인 메모리가 가지고 있는 값과 다르다면 값을 반영하지 않고 `false`를 리턴해 메인 메모리에 저장된 값을 읽어 2번으로 돌아간다.
   2. 기존 값이 메인 메모리가 가지고있는 값과 같으면 변경할 값을 반영하고 `true`를 리턴한다.






# 3. AtomicInteger

`AtomicInteger`는  멀티 쓰레드 환경에서 안전하게 정수값을 증가 또는 감수시킬 수 있는 기능을 제공한다. 내부적으로 `CAS(Compare-And-Swap)`을 사용해 원자성을 보장한다. 따라서 여러 쓰레드가 동시에 `AtomicInteger` 값을 조장해서 데이터의 일관성이 유지된다.

```java
public class AtomicIntegerExample {
    public static void main(String[] args) {

        AtomicInteger atomicInt = new AtomicInteger(0);

        //증가 연산
        int incrementedValue = atomicInt.incrementAndGet();
        System.out.println(incrementedValue);

        //감소 연산
        int decrementedValue = atomicInt.decrementAndGet();
        System.out.println(decrementedValue);

        //현재 값 가져오기
        int currentValue = atomicInt.get();
        System.out.println(currentValue);

        // 특정 값으로 설정
        atomicInt.set(10);
        System.out.println("New Value: " + atomicInt.get()); //10

        //CAS 연산
			  //특정값(10)과 현재값(10)이 일치할때만 20으로 업데이트 후 true 리턴 
				//반환값은 boolean
        boolean updated = atomicInt.compareAndSet(10, 20);
        System.out.println("Update Successful: " + updated);
        System.out.println("Current Value: " + atomicInt.get());

				// CAS 연산
				//값이 일치하면 새로운 값으로 업데이트, 값이 일치하지 않으면 현재값 유지
        // 반환값은 변경 이전의 값을 반환함. 즉 일치하지 않든 일치하든 이전 값 반환
        int oldValue = atomicInt.compareAndExchange(20, 30);
        System.out.println("Old Value: " + oldValue); //반환값
        System.out.println("Current Value: " + atomicInt.get());// 일치하면 30으로 업데이트
    }
}
```

이렇게 `AtomicInteger`를 사용하면 멀티쓰레드 환경에서 안전하게 정수값을 다룰 수 있다.

## 3.1 주요 메소드

- `AtomicInteger` : `AtomicInteger` 클래스의 생성자로 초기값을 할당 할 수 있다. 이때 값은 `valatile` 로 선언되었는데 그 이유는 `AtomicInteger` 은 원자적인 연산을 보장하는데 이때 연산을 하는 변수값들이 `valatile` 로 인해 항상 메모리에서 값을 확실하게 읽어올 수 있도록 해준다.

```java
		private volatile int value;

    /**
     * Creates a new AtomicInteger with the given initial value.
     *
     * @param initialValue the initial value
     */
    public AtomicInteger(int initialValue) {
        value = initialValue;
    }
```

- `compareAndSetInt(expectedValue, newValue)` : 첫번째 인자로 `예상하는 값`, 두번째 인자는  `새로운 값`을 말한다.  `예상하는 값`과 `현재값`이 일치해야 새로운 값으로 설정, 그렇지 않으면 아무런 작업도 수행하지 않는다. 이 과정은 원자적이기때문에 다른 쓰레드가 동시에 같은 연산을 해도 충돌이 발생하지 않는다.

```java
/**
     * Atomically sets the value to {@code newValue}
     * if the current value {@code == expectedValue},
     * with memory effects as specified by {@link VarHandle#compareAndSet}.
     *
     * @param expectedValue the expected value
     * @param newValue the new value
     * @return {@code true} if successful. False return indicates that
     * the actual value was not equal to the expected value.
     */
    public final boolean compareAndSet(int expectedValue, int newValue) {
        return U.compareAndSetInt(this, VALUE, expectedValue, newValue);
    }
```

- `compareAndExchange(expectedValue, newValue)`: `compareAndSetInt()` 와 마찬가지로 예상하는 값과 `현재 값`을 비교하지만. 반환값이 다르다. `compareAndSet`은 `boolean`을 반환하는데 비해 `compareAndExchange`는 값을 반환하는데 이때 반환값은 값이 일치하든 말든 무조건 이전값이다.

```java
/**
     * Atomically sets the value to {@code newValue} if the current value,
     * referred to as the <em>witness value</em>, {@code == expectedValue},
     * with memory effects as specified by
     * {@link VarHandle#compareAndExchange}.
     *
     * @param expectedValue the expected value
     * @param newValue the new value
     * @return the witness value, which will be the same as the
     * expected value if successful
     * @since 9
     */
    public final int compareAndExchange(int expectedValue, int newValue) {
        return U.compareAndExchangeInt(this, VALUE, expectedValue, newValue);
    }
```

더 다양한 메소드는 링크에서 확인 가능하다.

- [Java Development Kit Version 17 API Specification](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html)