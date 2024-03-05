# 📌ConcurrentHashMap과 SynchronizedMap의 특징과 차이점

동기화된 `HashMap`을 생성하는 방법에는 2가지가 존재한다. `SynchronizedMap` 을 사용하거나 `ConcurrentHashMap` 을 사용할 수 있다.

# 1.S**ynchronizedMap**

`SynchronizedMap` 은 자바 `Collections` 패키지에 속하며 `Map 인터페이스`를 구현한 클래스이다.

```java
    private static class SynchronizedMap<K,V>
        implements Map<K,V>, Serializable {
        @java.io.Serial
        private static final long serialVersionUID = 1978198479659022715L;

        @SuppressWarnings("serial") // Conditionally serializable
        private final Map<K,V> m;     // Backing Map
        @SuppressWarnings("serial") // Conditionally serializable
        final Object      mutex;        // Object on which to synchronize

        SynchronizedMap(Map<K,V> m) {
            this.m = Objects.requireNonNull(m);
            mutex = this;
        }

        public int size() {
            synchronized (mutex) {return m.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return m.isEmpty();}
        }
        public boolean containsKey(Object key) {
            synchronized (mutex) {return m.containsKey(key);}
        }

			//이하생략
}
```

`SynchronizedMap` 은 이름 그대로 모든 메소드에 `synchronized` 가 붙어있다. 따라서 락을 얻은 하나의 쓰레드만이 메소드를 실행할수있고  다른 쓰레드는 작업을 할 수 없다. 이 말은 동기화를 제공하지만  멀티쓰레드 상황에서 성능이 안 좋다. `SynchronizedMap`은 주로 `HashMap`을 동기화를 해야할때  사용했으나 `Java 5` 이후 더 효율적인 `ConcurrentHashMap` 이 등장해 잘 사용하지 않는다.

`synchronizedMap`을 다음과 같이 사용할 수있다. 주어진 코드에서 여러 쓰레드가 동시에 `put`를 호출하더라도 해당 메소드는 동기화 되어있으므로 안전하게 사용할 수있다.

```java
public class SynchronizedMapExample {
    public static void main(String[] args) {

        // 일반적인 HashMap 생성
        Map<String, Integer> hashMap = new HashMap<>();
        //synchrnoziedMap으로 래핑
        Map<String, Integer> synchronizedMap = Collections.synchronizedMap(hashMap);

        //람다를 이용해  Runnable 인터페이스 구현
        Runnable runnable = () -> {
            for (int i = 0; i < 5; i++) {
                synchronizedMap.put("key" + i, i); //동기화
            }
        };

        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start(); //쓰레를 생성하고 run 메서드 호출
        thread2.start();

        try {
            thread1.join(); //thread1 너 먼저해
            thread2.join(); //thread2 너 먼저행
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 동기화된 맵 출력
        System.out.println("Synchronized Map: " + synchronizedMap);
    }
}

```

# **2. ConcurrentHashMap**

`ConcurrentHashMap`은 `java.util.concurrent` 패키지에 속하는 클래스이다. 자바의 `java.util.concurrent` 패키지는 `자바 5`부터 추가된 패키지로, `동시성(Concurrency)`을 다루기 위한 다양한 클래스와 인터페이스를 제공한다. `ConcurrentHashMap`은 이 패키지에서 제공하는 동시성 `Map` 중 하나이다.

![concurrentHashMap](https://github.com/princenim/TIL/assets/59499600/b7c835a9-9252-40fd-a5b5-742fa8a63c5e)

`Map`을  `세그먼트(슬라이스)`단위로 나누어  각 세그먼트에 대해 독립적인 `락(segmented lock)`을 사용한다. 예를 들어 크기가 10인 `HashMap`이 존재할때 1번 쓰레드가 0번 인덱스의 값을 업데이트하고, 2번 쓰레드가 5번 인덱스를 업데이트한다고 했을때 이 작업은 서로 다른 쓰레드가 서로 다른 인덱스를 처리하고 있으므로 동시에 일어나도 상관없다. 이렇게 **`HashMap`을 세그먼트 단위로 나누어서 구역마다 `락(segmented lock)`을 사용하는 개념이 `ConcurrentHashMap` 이다**. 따라서 `SynchronizedMap` 보다 상대적으로 성능이 좋다.

반면에 읽기 작업을 수행할때 정확하지 않은 값을 주는 경우가 있다. 두 개의 쓰레드가 동시에 `ConcurrentHashMap`에 데이터를 추가하고 있다면, 다른 쓰레드에서 `size()` 메소드로 전체 데이터 개수를 확인하는 경우에는 두 쓰레드의 작업이 모두 완료되기 전까지 정확한 개수를 반영하지 않을 수 있다. 즉, 쓰기 작업이 끝나기 전까지는 `size()` 메소드가 미처 업데이트되지 않은 값을 반환할 수 있다.

`ConcurrentHashMap` 은 다음과 같이 사용할 수 있다.

```java
public class ConcurrentHashMapExample {
    public static void main(String[] args) {

        ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();

        Runnable runnable = () -> {
            for (int i = 0; i < 5; i++) {
                concurrentMap.put("key" + i, i);
            }
        };

        // 여러 스레드 생성 및 실행
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();

        try {
            thread1.join(); //thread1 너 먼저해랑
            thread2.join(); //thread2 너 먼저해랑
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // ConcurrentHashMap 출력
        System.out.println("ConcurrentHashMap: " + concurrentMap);
    }

}
```

# 3. ConcurrentHashMap와 SynchronizedMap의 차이점

`ConcurrentHashMap`은 각 세그먼트에 대해서 독립적인 락을 사용한다 따라서 여러 쓰레드가 다른 세그먼트에 대한 작업을 안전하게 수행할 수 있다. 반면에 `SynchronizedMap`는 락을 사용한다는 점은 같지만 하나의 쓰레드만이 락을 사용할 수있고 다른 쓰레드는 `blocking` 상태가 되므로 멀티 쓰레드 환경에서`ConcurrentHashMap`가 `SynchronizedMap`보다 성능이 더 좋다.