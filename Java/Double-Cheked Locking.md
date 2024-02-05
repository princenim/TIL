# 1. Double-Checked Locking

## 1.1 싱글톤 패턴

인스턴스가 오직 1개만 생성되는 패턴을 말한다. 다시 말해 클래스에 대해 인스턴스는 단 한번만 생성된다. 따라서 해당 클래스의 객체가 전역적으로 유일하게 존재한다. 생성자를 private로 만들어 외부에서 생성자를 직접호출할수 없고 `getInstance()`를 통해서만 인스턴스를 얻을 수 있다. 

```java
public class MySingleton {

    // 정적 멤버 변수로 유일한 인스턴스를 저장
    private static MySingleton instance;

    // 생성자를 private으로 선언하여 외부에서 인스턴스 생성을 막음
    private MySingleton() {
        // 생성자 코드
    }

    // 인스턴스에 접근할 수 있는 정적 메서드
    public static MySingleton getInstance() {
        // 인스턴스가 없는 경우에만 생성
        if (instance == null) {
            instance = new MySingleton();
        }
        return instance;
    }
}
```

하지만 위의 코드는 멀티 쓰레드 환경에서 Thread-safe하지않다. 멀티 쓰레드 환경에서 동시에 실행된다면 인스턴스가 두번 생성될 수 도있다. 

그렇다면 어떻게 해결해야할까?

### synchronized

먼저  `synchronized` 방법을 사용할수 있다. 

```java
public class MySingleton {
    private static MySingleton instance;

    private MySingleton() {}//생성자

    public static synchronzied MySingleton getInstance() {
      if(instance  == null) {
         instance  = new MySingleton();
      }
      return instance;
    }
}

```

하지만 이 방법은 사용하는 쓰레드에 락이 걸려 다른 쓰레드는 락이 풀릴때까지 기다려야하므로 성능저하를 일으킨다.
이 방법보다 더 좋은 성능을 보장하는 방법이 있다. 바로 `double-checked locking` 이다. 

## 1.2 double-checked locking pattern

`Double-Checked Locking`은 소프트웨어 디자인 패턴 중 하나로 주로 싱글톤 패턴에서 멀티쓰레드 환경에서의 성능을 높이기 위해서 사용되는 방법이다. 이미 초기화된 경우에는 락을 획득하지 않고 바로 반환해하고, 초기화 되지 않은 경우에만 락을 획득해 초기화를 진행하는 방법으로 성능을 높인다.

```java
public class MySingleton {
    private static volatile MySingleton instance;

    private MySingleton() { //생성자

		}

    public static MySingleton getInstance() {
        if (instance == null) {  // 첫 번째 체크
            synchronized (MySingleton.class) { //락을 획득 - synchronized 키워드를 사용한 블럭을 의미 
                if (instance == null) {  // 두 번째 체크
                    instance = new MySingleton();
                }
            }
        }
        return instance;//반환 
    }
}
```

1. **첫번째 체크(First check)** : 초기화과 되었는지 확인하기 위해 먼저 락없이 검사를 한다. 
2. **락을 획득해 두번째 체크(Acquire Lock and Second check)** : 첫번째 체크가 통과하면 락을 획득한 후 초기화가 되었는지를 다시 한번 확인한다. 이 시점에 다른 쓰레드가 이미 락을 획득해 초기화를 진행한 경우 락을 획득하지 않고 빠르게 반환한다. 
3. **인스턴스 초기화(Initialization of Instance)** : 두번째 체크에서 초기화가 필요한 경우에만 초기화를 진행한다. 
4. **락 해제(Release Lock)** : 초기화가 완료되면 락을 해제하고 인스턴스를 반환한다. 

또한 중요하게 봐야할 부분은 `volatile` 키워드이다.  위의 코드를 보면 `volatile` 키워드를 확인할 수 있는데 이 키워드를 사용하여 인스턴스 변수에 대해 가시성을 보장할 수 있다. **가시성이란 하나의 쓰레드에서 수행한 작업의 결과가 다른쓰레드에서 올바르게 볼수 있는 정도를 나타낸다. 좀더 쉽게말하면 쓰레드가 변경한 값이 메인 메모리에 저장되지 않아서 다른 쓰레드가 이 값을 볼수 없는 상황을 가시성 문제라고한다.** 

만약에 `volatile`을 사용하지 않는다면 첫번째 쓰레드가 인스턴스를 생성하고 `synchronized` 블록을 벗어났는데 두번째 쓰레드가 `synchronized` 블록에 들어와서 null체크를 하는 시점에서(두번째 체크) 첫번째 쓰레드가 생성한 인스턴스가 캐시에만 존재하고 메인 메모리에는 존재하지 않을때 두번째 쓰레드는 인스턴스를 또 생성하기 때문에 인스턴스를 레퍼런스하는 변수에 `volatile`을 꼭 사용해야한다.