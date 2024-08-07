
# **📌** Atomic과 CAS 알고리즘

# 1. Atomic

멀티 쓰레드 환경에서 동시성 이슈를 다루기 위해 동기화가 필요한데, 이를 위해 `synchronized` 키워드를 사용하는 경우 해당 블럭 전체를 락으로 잡아 다른 쓰레드들이 대기 상태에 놓이게 된다. 이는 블럭에 접근하는 쓰레드들이 `blocking`상태가 되어 자원을 기다리게 하며, 이로 인해 성능 저하가 발생할 수 있다.

멀티 쓰레드 환경에서 동시성 이슈를 다루기 위해 동기화가 필요한데, `synchronized` 키워드를 사용해서 동기화를 구현할 수 있다. `synchronized`를 사용하면 해당 블럭 전체를 lock(점유)하기 때문에 다른 쓰레드는 아무런 작업을 하지 못하고 기다리게된다. 다시말해 이때 블럭에 접근하는 쓰레드들은 `block` 상태가 된다. 즉 쓰레드들이 `blocking` 되는 과정과 다시 `resuming` 되는 과정에서 시스템의 자원을 사용하게되는데. 바로 이 부분에서 성능 저하가 발생한다.

이러한 성능 문제를 해결하기 위해 `non-blocking` 방식의 `Atomic` 클래스를 사용할 수 있습니다.

`Atomic`은 '원자성'을 의미하며, 작업이 더 이상 쪼개어질 수 없는 가장 작은 단위로, 다른 작업에 의해 간섭되지 않는 것을 의미합니다. 이는 여러 쓰레드 간의 충돌을 방지하고, 중간에 간섭되지 않음을 보장한다.

```java

Integer a = 10;
a++;
```

예를 들어  `a`가 라는 값이 있을때, `a` `플러스` `1`을 하려고 하면 `CPU` 입장에서 총 몇번의 작업을 실행해야할까?`CPU` 입장에서 총 3번 작업이 실행해야한다.

1. 값을 읽고 (read)
2. 값을 증가시키고
3. 값을 저장한다. (write)

근데 만약 값을 저장하려고 하는 시점에 다른 쓰레드가 `a` 값에 12이라고 저장을 했다면 어떻게 해야할까? 내가 값을 저장하려고 했던 시점에서는 `a`의 값이 `10`이었는데 메모리에 있는 값은 `12`가 되었다. 그렇다면 값은 `11`이 되어야할까 `13`이 되어야할까?

즉, 해당 작업이 원자적으로 실행되지 않는다면 다른 쓰레드에 의해 값이 변경될수 있으므로 작업은 원자적으로 실행되어야한다.

이러한 문제를 `Atomic` 클래스를 이용하여 해결할수있는데  `Atomic`의 핵심원리가 바로 `CAS(Compare And Swap)알고리즘`이다.  자바에서는 `java.util.concurrent.atomic` 패키지에서 원자적인 연산을 지원하는 클래스를 제공하고 이 클래스들은 모두 `CAS(Compare And Swap)알고리즘` 로 구현되어있다. 그리고 `Atomic` 방식은 락을 획득하고 반납하는 과정이 없기 때문에 `synchronized`보다 훨씬 성능이 좋다.

# 2. CAS 알고리즘

![ㅊㅁㄴ](https://github.com/princenim/TIL/assets/59499600/75ef1587-b711-4711-a658-0b199c0925cd)

CAS 알고리즘의 동작 원리는 다음과 같다.

1. 인자로 `기존 값(Compared value)`와 `변경할 값(Exchanged value)`를 전달한다.
2. `기존값(Compared value)`이 `현재 메모리가 가지고 있는 값`과 같다면 `변경할 값(Exchanged value)`을 반영해 `true`를 반환한다.
3. 반대로 `기존 값(Compared value)`이 `현재 메모리가 가지고 있는 값(destination)`과 다르다면 값을 반영하지 않고 `false`를 반환한다.

이해가 잘 가지 않으니 그림으로 그리면 다음과 같다.

![제목 없는 그림](https://github.com/princenim/TIL/assets/59499600/5ce4ca8f-b007-450d-a68a-57fc89d74c6d)

만약에 a+5를 하려고 할때 CAS 는 다음과 같이 동작한다.

1. 먼저 메모리에 저장된 값 3을 가져와서 CPU 캐시에 저장한다.
2. 가져온 값을 가지고 3+5를 해서 값을 더한다.
3. 더한 값 8을 CPU 캐시에 저장한다.
4. 이제 기존에 메모리에서 읽어온 값 3과 더하려는 값을 8을 보내서 메모리에 저장된 값과 기존 메모리에서 읽어온 값(CPU 캐시에 저장된 값)이 같을 때 8로 업데이트 한다.

**이때 만약에!** 다른 쓰레드에서 먼저 연산을 해서 memory의 값이 10으로 업데이트가 되었다면 기존 쓰레드가 온값 3과 메모리에 저장된 값 10이 다르므로 업데이트를 하지 않는다.

이렇게 여러 쓰레드가 동시에 CAS를 시도하더라고 오직 하나의 쓰레드만이 CAS를 성공시키고 나머지는 재시도해야하므로 CAS 알고리즘은 멀티쓰레드 환경에서 공유자원을 안전하게 업데이트시킨다. 즉, CPU 캐시에 저장된 값(작업을 시작할때 읽은 값)과 메인 메모리에 저장된 값(작업이 끝났을때 읽은 값)을 비교해 일치하는 경우 새로운 값으로 교체하고 , 일치하지 않으면 다시 재시도를 함으로써  멀티 쓰레드 환경에서 데이터의 일관성을 유지할 수 있다.

그리고 CAS가 성립하려면 핵심은 값을 비교하고, 메모리의 값을 업데이트 하는과정이 원자적이어야한다.

![cas](https://github.com/princenim/TIL/assets/59499600/4a119b34-86d8-4b9d-a2d0-7f98696a510e)

위의 그림에서 알수있듯이 각 쓰레드는 독립적인 CPU 캐시를 가진다. 따라서 쓰레드가 사라지면 해당 쓰레드의 CPU 캐시도 사라진다.

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

멀티쓰레드 환경에서 숫자를 처리할때 `synchronized` 보다 `AtomicInteger`를 많이 사용한다. 왜나하면 멀티 쓰레드 환경에서 요청이 동시에 갈 가능성은 매우 적기때문이다.   `synchronized`를 사용할 수 있지만 요청이 겹치든 안 겹치든 무조건 쓰레드는 락을 획득해야하는데 이 비용이 매우 크다. 따라서 `AtomicInteger`를 활용해 필요할때만 여러번 재시도하는 방법이 훨씬 더 좋은 방법이다.

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


[Java Development Kit Version 17 API Specification](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html)