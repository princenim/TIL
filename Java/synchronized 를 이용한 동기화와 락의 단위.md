# 📌synchronized를 이용한 동기화와 락의 단위

# 1. 쓰레드의 동기화

싱글쓰레드의 프로세스의 경우 프로세스 내에서 단 하나의 쓰레드만 작업하기 때문에 프로세스의 자원을 가지고 작업하는 문제가 없지만, 멀티 쓰레드 프로세스의 경우 여러 쓰레드가 같은 프로세스 내의 자원을 공유해 작업하기 때문에 서로의 작업에 영향을 준다. 만약에 `쓰레드 A`가 작업하던 도중에 다른 `쓰레드B`에게 제어권이 넘어갔을 때 `쓰레드 A`가 작업하던 공유데이터를 `쓰레드 B`가 변경하였다면 다시 `쓰레드 A`가 제어권을 받아서 나머지 작업을 마쳤을 때 원래 의도했던 것과는 다른 결과를 얻을 수 있다. 이러한 일을 방지하기 위해 한 쓰레드가 특정 작업을 끝마치기 전까지 다른 쓰레드에 의해 방해받지 않도록 하는 것이 필요하다. 그래서 도입된 개념이 `임계영역(critical section)`과 `잠금(락, lock)`이다.

**임계영역은 둘 이상의 쓰레드가 공유 자원에 접근할 때 오직 한 쓰레드만 접근을 보장하는 영역을 말한다.** 또한 모든 객체는 `락(lock)`을 갖고 있다. 모든 객체가 갖고 있으니 고유락(Intrinsic Lock)이라고 하며 모니터 락 또는 모니터 라고 부르기도한다.

공유 데이터를 사용하는 코드 영역을 임계영역으로 지정하고 공유 데이터(객체)가 가지고 있는 `Lock`을 획득한 단 하나의 쓰레드만 이 영역 내의 코드를 수행할 수 있게 한다. 그리고 해당 쓰레드가 임계영역 내의 모든 코드를 수행하고 벗어나서 `lock`을 반납해야만 다른 쓰레드가 반납된 `lock`을 획득해 임계영역의 코드를 수행할 수 있게 된다. **이렇게 한 쓰레드가 진행 중인 작업을 다른 쓰레드가 간섭하지 못하도록 막는 것을 쓰레드의 `동기화(syncronization)`라고 한다.**

자바에서는 `synchronized` 블럭을 이용한 동기화를 지원했지만, `JDK 1.5`부터 `java.util.concurrent.locks` 와 `java.util.concurrent.atomic` 을 통한 다양한 동기화 방식을 지원한다

### 동시성

동시성은 하나의 코어에서 여러 쓰레드를 번갈아 작업하면서 처리하는 방식이다. 즉 동시에 하는 것처럼 보이는 것을 말한다.  따라서 멀티쓰레드에서 공유하는 영역이 많아 `context switching` 비용이 작다는 점이 장점이 있다. 하지만 자원을 공유하기 때문에 단점도 존재한다. 그게 바로 동시성 이슈이다.

여러 가지 쓰레드가 “동시에” 하나의 자원에 공유하고 있기 때문에 `경쟁상태(race condition)`과 같은 문제가 발생한다.  이런 동시성 문제를 해결하는 방법에는 대표적으로 **동기화, volatile 키워드 , 원자적 연산(Atomic)** 이 있는데 여기서는 동기화에 대해 이야기할 것이다.

# 2. Synchronized를 이용한 동기화

동시성 문제를 해결하기 위한 `synchronized` 키워드를 이용한 동기화 방식에는 2가지가 있다.

### **2.1 메서드 영역 전체를 임계영역으로 지정**

쓰레드는 `syncrhonized` 메소드가 호출된 시점부터 해당 **메소드가 포함된 객체의 lock을 얻어** 작업을 수행하다가 메소드가 종료되면 lock을 반환한다.

```java
public syncronized void calcSum(){ //메서드 전체가 임계영역

}
```

### **2.2 특정한 영역을 임계 영역으로 지정**

메소드 내에 코드 일부를 `블럭{}` 으로 감싸고 블럭 앞에 `synchronized(참조변수)` 를 붙이는 방법으로  이때 참조변수는 락을 걸고자 하는 객체를 참조하는 것이어야 한다. 이 블럭의 영역 안으로 들어가면서부터 쓰레드는 지정된 객체의 `lock`을 얻게 되고 이 블럭을 벗어나면 `lock`을 반환한다.

```java
public void method(...) {
		syncronized(객체의 참조변수){
	}
}

```

그리고 이 `synchronized` 블록은 **2가지의 사용방법**이 존재한다.

### **synchronized(this)**

```java
public class Block1 {

    public static void main(String[] args) {
        Block1 block = new Block1();

        //람다식으로 쓰레드1 생성
        Thread thread1 = new Thread(
                () -> {
                    System.out.println("쓰레드 시작1 :" + LocalDateTime.now());
                    block.syncBlockMethod1("쓰레드1");
                    System.out.println("쓰레드 1 종료" + LocalDateTime.now());
                }
        );

        //람다식으로 쓰레드2 생성
        Thread thread2 = new Thread(
                () -> {
                    System.out.println("쓰레드 시작2 :" + LocalDateTime.now());
                    block.syncBlockMethod2("쓰레드2");
                    System.out.println("쓰레드 2 종료" + LocalDateTime.now());
                }
        );

        thread1.start();
        thread2.start();

    }

    private void syncBlockMethod1(String msg) {
        synchronized (this) { //this는 Block1 의 객체를 말함.
            System.out.println(msg + "의 syncBlockMethod1 실행 중" + LocalDateTime.now());

            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void syncBlockMethod2(String msg) {
        synchronized (this) { //this는 Block1 의 객체를 말함.
            System.out.println(msg + "의 syncBlockMethod2 실행 중" + LocalDateTime.now());

            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

```

<img width="712" alt="1" src="https://github.com/princenim/TIL/assets/59499600/88810f0d-0e7a-4e9f-ab19-645393245f6b">

```java
//결과
쓰레드 시작2 :2024-02-08T16:45:23.262817
쓰레드 시작1 :2024-02-08T16:45:23.262817
쓰레드2 의 syncBlockMethod2 실행 중2024-02-08T16:45:23.264809
쓰레드 2 종료2024-02-08T16:45:28.282979
쓰레드1 의 syncBlockMethod1 실행 중2024-02-08T16:45:28.282979
쓰레드 1 종료2024-02-08T16:45:33.284676
```

먼저 `this`는 클래스의 인스턴스 메모리 주소를 가리킨다. 즉 클래스가 다른 객체를 생성했을때는 this는 서로 다른 객체를 가리킨다.   `sybchronized` 블록에 `this`를 사용하면 모든 `synchronized` 블록에 하나의 객체에 같은 락을 사용하므로 한 `synchronized` 블럭이 끝나고 락이 반환될때까지 다른 모든 `synchronized`  블록은 사용할 수 없다.

따라서  블록마다 다른 락을 갖게 하는 방법이 `synchronized(Object)`방식이다.

### **synchronized(Object)**

```java
public class Block2 {

    private final Object o1 = new Object();
    private final Object o2 = new Object();

    public static void main(String[] args) {
        Block2 block = new Block2();

        //람다식으로 쓰레드1 생성
        Thread thread1 = new Thread(
                () -> {
                    System.out.println("쓰레드 시작1 :" + LocalDateTime.now());
                    block.syncBlockMethod1("쓰레드1");
                    System.out.println("쓰레드 1 종료" + LocalDateTime.now());
                }
        );

        //람다식으로 쓰레드2 생성
        Thread thread2 = new Thread(
                () -> {
                    System.out.println("쓰레드 시작2 :" + LocalDateTime.now());
                    block.syncBlockMethod2("쓰레드2");
                    System.out.println("쓰레드 2 종료" + LocalDateTime.now());
                }
        );

        thread1.start();
        thread2.start();

    }

    private void syncBlockMethod1(String msg) {
        synchronized (o1) { //o1이라는 이름의 객체 락을 가짐
            System.out.println(msg + " 의 syncBlockMethod1 실행 중" + LocalDateTime.now());

            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void syncBlockMethod2(String msg) {
        synchronized (o2) { //o2라는 이름의 객체 락을 가짐
            System.out.println(msg + " 의 syncBlockMethod2 실행 중" + LocalDateTime.now());

            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

<img width="711" alt="2" src="https://github.com/princenim/TIL/assets/59499600/461b8e26-b719-4360-9fed-66dbd34b22d5">

이렇게 `this`가 아닌 객체를 따로 만들어 블럭이 갖는 객체의 락을 다르게 해주면 `synchronized 블록`은 서로 동시에 실행할 수 있다. 그리고 이때 만드는 `o1`, `o2`는 락을 위해서만 사용하는 임시객체이므로 어느것으로 만들어도 상관없다.

즉 정리하면 `this` 방식은 모든 블럭에 lock이 걸리므로 상황에 따라 비효율적일 수있다. 하지만 Object 방식은 블록마다 다른 lock을 걸리게 하여 효율적으로 코드를 작성할 수 있다.

# 3. 락

위에서 계속 락에 대해 이야기했는데 락에 대해 좀더 자세히 알아보자.

`synchronized 블럭`이라는 하나의 영역에서 한 쓰레드가 작업을 하려고 하면 다른 쓰레드는 접근할 수 없다.

즉 이말은 누군가가 하나의 쓰레드만 접근할 수 있도록 컨트롤을 해야줘야한다는 의미이다.

이를 자바에서는 `lock(intrinsic lock)` 이라고 하며 `monitor lock(모니터 락)` 또는 `monitor(모니터)`이라고 부르기도한다.  이 락은 모든 객체에 하나씩 존재한다.

좀 더 쉽게 생각하면 화장실을 생각하면 된다. 화장실(임계영역)에 들어가면 문을 자물쇠(락)로 잠궈야지 다른 사람(쓰레드)이 못 들어온다. **😦**

## 3.1 락의 단위

위 설명한 락은 모두 `Object 레벨의 락`이다. 하지만 또다른 `클래스 레벨의 락`이 존재한다.

1. **Object level lock**

```java
private void syncBlockMethod1(String msg) {
        synchronized (this) { //this level
    }
}
```

클래스의 모든 인스턴스가 각자의 `lock`를 가진다.

2. **Class level lock**

```java
private void syncBlockMethod1(String msg) {
        synchronized (Block1.class) { //Class level
    }
}
```

클래스의 인스턴스가 여러개 있어도 하나의 lock을 가진다.  클래스 하나당 하나의 lock을 가질 수 있다.   예를들어 위의 예제에서 `Block1` 이라는 클래스의 객체를 수십개 만들었을 때 모든 객체에 대해서 동일한 락을 하나만 주고싶을 때 사용한다. 즉  클래스 레벨이 객체레벨보다 더 범위가 크다 .

```java
public class Block3 implements Runnable {

    @Override
    public void run() {
        syncBlockMethod1();
    }

    private void syncBlockMethod1() {
        synchronized (Block3.class) { //Class level lock

            System.out.println("in block " + Thread.currentThread().getName() + " 시작");

            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("in block " + Thread.currentThread().getName() + " 끝");
        }
    }

    public static void main(String[] args) {
        Block3 block1 = new Block3();
        Thread thread1 = new Thread(block1);
        Thread thread2 = new Thread(block1);

        Block3 block2 = new Block3();
        Thread thread3 = new Thread(block2);

        //쓰레드 이름
        thread1.setName("thread1");
        thread2.setName("thread2");
        thread3.setName("thread3");

        thread1.start();
        thread2.start();
        thread3.start();

    }
}
```

```java
in block thread1 시작
in block thread1 끝
in block thread3 시작
in block thread3 끝
in block thread2 시작
in block thread2 끝
```

위의 코드는 `클래스 레벨의 락`을 사용한 예제이다.  `Block3`이라는 클래스의 인스턴스 `block1`,`block2` 2개를 생성했지만 락이 클래스 레벨로 설정되어 모두 같은 락을 갖게되어서 순서대로 시행되는 결과가 나왔다.

만약에 위의 코드를 단순히 `this`로 바꾸면 인스턴스마다 다른 객체를 갖게되어 서로 다른 락을 갖게되고 순서대로 실행하지 않는다.