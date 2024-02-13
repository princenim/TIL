# 📌 Concurrent 패키지의 atomic과 locks

# 1. java.util.concurrent

자바는 `java.util.concurrent` 라는 패키지가 존재한다. 이 패키지는 동시성(concurrency) 프로그래밍에 도움을 주는 클래스와 인터페이스를 제공한다.

[Java Development Kit Version 17 API Specification](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/package-summary.html)

- `java.util.concurrent.atomic` : 동기화가 되어있는 변수를 제공한다.
- `java.util.concurrent.locks` : 상호 배제를 사용할 수 있는 클래스를 제공한다.
- `Executors` : 쓰레드 풀 생성, 쓰레드 생명주기 관리, Task 등록과 실행 등을 간편하게 처리할 수 있다.
- `queue`: thread-safe한 FIFO 큐를 제공한다.
- `Synchronizers` :  특수한 목적의 동기화를 처리하는 5개의 클래스를 제공한다. (Semaphroe, CountDownLatch, CyclicBarrier, Phaser, Exchanger)

# 2.  java.util.concurrent.atomic

`atomic` 패키지는 `CAS(Compare and swap)` 방식으로 원자성을 보장하는 클래스들을 제공한다. `AtomicBoolean`, `AtomicInteger`, `AtomicLong`, `AtomicReference`와 같은 클래스들을 제공한다.

더 자세한 설명은 이 글에서 확인 가능하다.

- [Atomic과 CAS 알고리즘](https://github.com/princenim/TIL/blob/master/Java/Atomic%EA%B3%BC%20CAS%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98.md).

## 2.1 atomic과 synchronized의 차이점

- `synchronized` 를 사용하면 락을 갖지 못한 쓰레드는 락을 획득하기를 기다려야하므로 성능 저하를 일으킬 수 있다. 하지만 `atomic`은 원자적 연산을 통해 락을 획득하지 않고도 쓰레드간의 안전한 동기화 연산이 가능하다. 성능상으로는 atomic이 훨씬 빠르다.
- `synchronized` 는 블록 또는 메소드 단위에서 사용되나 `atomic`은 주로 원자적 연산을 제공하는 변수나 특정 데이터 타입에 대해 사용된다. 즉 전체 블록이 아닌 일부분만 동기화한다.

# 3.  java.util.concurrent.locks

<img width="342" alt="스크린샷 2024-02-13 오후 4 01 11" src="https://github.com/princenim/TIL/assets/59499600/ff7343bb-f051-45ef-ba73-b47909e1664c">


자바에서 `synchronized` 블럭 외에도 다른 방법으로 동기화가 가능하다.그 중 하나가  `java.util.concurrent.locks` 가 제공하는 lock 클래스를 이용하는 것이다. 이 패키지를 사용하면 `synchronized` 를 사용하는 것 보다 더 유연하게 사용가능하다.  구현체를 크게 보자면 `Lock` 인터페이스를 구현하는 `ReentrantLock`과 `ReadwriteLock`을 구현하는 `ReentrantReadWriteLock`이 있다.

## 3.1 락의 종류

먼저 락의 종류를 알아보자. 락에는 `명시적 락`과 `암묵적 락`이 있다.

- **암묵적 락(Implicit Lock)** : `synchronized` 키워드를 사용하는 방식을 말하며 키워드를 사용하면 해당 영역이 암묵적으로 락 되어 여러 쓰레드 간의 동기화를 제공한다. 다시말해 락이 자동으로 획득 메소드나 블록의 실행이 끝날때 자동으로 해제된다.

```java
public synchronized void synchronizedMethod() {
    // 락이 보호하는 코드 영역
}
```

- **명시적 락(Explicit Lock):** 개발자가 직접 락을 획득하고 해제하는 락이다. `ReentrantLock` 가 대표적인 명시적 락이다. 개발자는 `lock()` 으로 락을 획득하고 `unlock()`으로 메소드로 락을 해제할 수 있다.

```java
ReentrantLock lock = new ReentrantLock();

lock.lock();  // 명시적 락 획득
try {
    // 락이 보호하는 코드 영역
} finally { //무조건 실행하는 구문
    lock.unlock();  // 명시적 락 해제
}
```

## 3.2 **ReentrantLock**

`java.util.concurrent.locks` 패키지에서 제공하는 명시적 락이다.

```java
public class ReentrantLock implements Lock, java.io.Serializable {

}
```

`ReentrantLock`은 이름 그대로 **재진입이 가능한 락**을 말한다. 재진입이 가능하다는 것은 같은 쓰레드에서 이미 획득한 락을 여러번 다시 획득할 수 있다는 의미이다. 동시에 서로 다른  여러개의 락을 얻을 수 있다는 의미는 아니다.

```java
ReentrantLock lock = new ReentrantLock();

lock.lock();  // 락 획득
try {
    // 락이 보호하는 코드 영역
} finally {
    lock.unlock();  // 락 해제
}
```

- `lock()`:  락을 잠근다.
- `unlock()` :  락을 해지한다. `unlock()`은 try 블럭 내에서 어떤 일이 발생해도 finally 블럭에 있어서 락이 풀리지 않는 일은 발생하지 않는다.
- `isLocked()` : 락이 잠겼는지 확인한다.
- `tryLocK()` : 다른 쓰레드에 의해 락이 걸려 있으면 락을 얻으려고 기다리지 않는다. 또는 지정된 시간만큼만 기다린다. 락을 얻으면 true 반환, 얻지 못하면 false를 반환한다.
- `tryLock()` : 해당 메소드를 사용해 락을 시도하고 다른 쓰레드에 의해 이미 락이 걸려있으면 락을 얻으려고 기다리지 않는다. `lock()`은 락을 얻을때까지 쓰레드를 블락시키므로 쓰레드의 응답성이 나빠질 수 있다. 따라서 응답성이 중요한 경우 `tryLock()`을 지정된 시간동안 락을 얻지 못하면 다시 작업을 시도할 것이지 포기할 것인지 정하는 것이 좋다.

### **ReentrantLock의 Condition**

`wait()`과 `notify()`는 `waitng pool`에 있는 쓰레드를 구분하지 못하고 랜덤으로 한 쓰레드에게 통지를 한다. 이런 문제점을 해결할수있는 방법이 `Condition`이다.

`Condition`은 이미 생성된 락으로부터 `new Condition()`을 호출해 생성할 수 있다  `wait()`과 `notify()`는 각각 `await()`과 `signal()`이라고 생각하면된다. `await()`은 실행중이던 쓰레드를 멈추고 기다리게한다. 그리고 `signal()`은 기다리고 있는 다른 쓰레드를 깨우는 역할을 한다.

```java

public class ConditionExample {
    private final ReentrantLock lock = new ReentrantLock(); //락 생성
    private final Condition condition = lock.newCondition(); // 락 컨디션 생성
    private boolean isDataReady = false; 

    public static void main(String[] args) {
        ConditionExample example = new ConditionExample();

        // Consumer 스레드 실행
        new Thread(() -> {
            try {
                example.consumeData();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // Producer 스레드 실행
        new Thread(() -> {
            example.produceData();
        }).start();
    }

    public void produceData() {
        lock.lock();//락 획득
        try {
            // 생산 작업 수행
            System.out.println("Producing data...");
            // 생산이 완료되었음을 표시
            isDataReady = true;
            // 대기 중인 스레드에 신호를 보냄
            condition.signal();
        } finally {
            lock.unlock(); //해당 코드에서는 lock이 해지되지않을 수있음. 
        }
    }

    public void consumeData() throws InterruptedException {
        lock.lock(); //락 획득
        try {
            // 데이터가 준비될 때까지 대기
            while (!isDataReady) {
                System.out.println("Waiting for data...");
                condition.await();//실행중이던 쓰레드를 멈추고 락을 기다림
            }
            // 데이터 소비 작업 수행
            System.out.println("Consuming data...");
        } finally {
            lock.unlock(); //락을 얻고 락을 해지
        }
    }
}
```

여기서 `lock`이 2개인것을 확인할 수 있는데 이는 같은 락이다. `ReentrantLock`가 재진입락이기 때문에 같은 락을 공유하고 있으며 여러분 중첩된 형태로 락을 획득할 수 있다. 하지만 락은 하나를 공유하기 때문에 다른 쓰레드가 반납하기 전까지 락을 획득 할 수 없다.

### 공정성

`ReentrantLock` 은 두 종류의 공정성을 지원한다. 불공정 방법과 공정방법이다. 이는 생성자에서 설정할 수 있다.

```java
ReentrantLock();//기본 생성자
ReentrantLock(boolean fair);//공정성 설정 가능
```

매개변수에 `true`을 주면 해당 락이 공정하게 동작하는데 이는 락을 요청한 순서대로 락을 부여하는 방식을 따른다. 공정한 락의 특징은 다음과 같다.

- **먼저 온 것이 먼저 서비스를 받는다. (First Come First Served)** : 락을 요청한 순서대로 락이 부여된다.
- **대기열 순서를 중요시한다** -  락을 얻지 못한 쓰레드는 대기열에서 기다린다.
- **공평성** - 우선순위나 특별한 요소없이 모든 쓰레드에게 공평하게 락을 양보한다.

즉 말 그대로 특정 쓰레드나 특정 조건이 없이 모든 쓰레드에게 기회가 동등하게 주어진다는 의미이다. 하지만 `true`로 공정하게 처리하면 어떤 쓰레드가 가장 오래 기다렸는지 확인하는 과정을 거쳐야하므로 성능이 떨어진다. 기본값은 `false` 로 대기 중인 쓰레드가 적은 경우에는 성능 향상을 위해 공평하지 않은 락을 사용하는 것이 더 좋다.

## 3.2 synchronized와 ReentrantLock의 차이점

1. `synchronized`는 블럭 구조를 사용하기 때문에 하나의 메소드 안에서 임게영역의 시작과 끝이 존재해야한다. 하지만 `ReentrantLock` 는 `lock()`과 `unlock()`으로 시작과 끝을 명시하기 때문에 임계영역을 여러 메소드에 나눠서 작성할 수 있다.
2. `synchronized` 는 동기화가 필요한 곳을 블럭으로 감싼다. 따라서 여러 쓰레드가 경쟁 상태에 있을 때 어떤 쓰레드가 먼저 진입할지 순서를 보장하지 않는다. (= 암시적 락)  `ReentrantLock` 은 `lock()`과 `unlock()` 순으로 메소드를 호출함으로써 어떤 쓰레드가 먼저 `lock()`를 획득하게 될지 순서를 지정할 수 있다. (= 명시적 락)
3. 공정성을 `ReentrantLock` 은 제공하나 `synchronizd`는 제공하지 않는다.
4. `tryLock()` 메소드의 제공 여부이다. `tryLock()` 은 락을 획득할 수 있을지 없는지 파악한다. 시간설정도 가능하므로 계속해서 다른 쓰레드를 기다리지 않아도 된다.