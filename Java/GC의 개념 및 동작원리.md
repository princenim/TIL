# 📌 GC (Garbage Collection)

## 1. GC란?

**GC(Garbage Collector, Garbage Collection)는 JVM의 힙 영역에서 더 이상 참조되지않는 객체를 메모리에서 해제시켜주는 장치이다.** JVM 내에서 자동으로 동작하기 때문에 Java는 개발자가 특별한 경우가 아니면 메모리 관리를 개발자가 직접해주지 않아도 된다.

![gc](https://github.com/princenim/TIL/assets/59499600/07761a59-bf26-4be1-be52-dca536c38d7f)

```java
Person person = new Person();  //참조변수 person이 객체를 가리킴
person.setName("hong")
person = null; //참조변수 person이 null로 더 이상 객체를 참조하지 않음.
```

위의 코드에서 person 객체는 더 이상 참조받지않고, 사용되지 않아서 Garbage가 되었다. 따라서 자바는 이러한 메모리 누수를 방지하기 위해 Garbage Colloctor를 이용해 주기적으로 메모리를 청소한다.

## 2. Young 영역과 Old 영역

JVM의 Heap영역은 처음 설계될 때 2가지 조건으로 설계되었다.

‘**Weak Generational hypothesis’**

- **대부분의 객체는 금방 접근 불가능한 상태(Unreachable)가 된다.**
- **오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.**

다시 말하면 객체는 대부분 일회성이며 보통 새로운 객체에서 오래된 객체를 참조한다는 것이다. 이러한 가정에서 Heap영역을 물리적으로 2개로 나누었는데 그게 Young 영역과 , Old 영역이다.

![heap ](https://github.com/princenim/TIL/assets/59499600/28b664d5-712f-4242-8fd3-2441925b32f8)

### 2.1 Young Generation(Young 영역)

새로운 객체가 할당되는 영역을 말한다. 대부분의 객체가 금방 *Unreachable*한 상태가 되서 생성되고 금방 사라진다. 이 영역에 대한 GC를 **Minor GC**라고 한다.

### 2.2 Old Generation(Old 영역)

Young 영역에서 reachable한 상태를 유지해 살아남은 객체가 복사된 영역이다.
복사되는 과정에서 대부분 Young 영역보다 크게 할당된다. 이 old 영역에 대한 GC를 **Major GC라**고 한다.

![car table](https://github.com/princenim/TIL/assets/59499600/e693f4aa-8f21-44ae-87ac-765fa775bb24)

그렇다면 예외적으로 Old 영역에서 Young 영역의 객체를 참조하는 경우는 어떻게 해야할까? 이러한 경우를 대비해 **Old 영역에는 512bytes의 덩어리로 되어있는 카드 테이블이 있다.** 카드 테이블은 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 관련된 정보가 생긴다. 따라서 Minor GC를 실행할때 Old 영역에 있는 모든 객체의 참조를 확인하지 않고, 이 카드 테이블만 확인해서 GC대상인지 식별한다.

## 3. GC의 동작원리

Young 영역과 Old 영역은 위의 그림으로 보듯이 서로 다른 메모리 구조를 가지고 있다. 세부적인 동작은 다르지만 하지만 기본적으로 다음과 같은 GC의 공통적인 단계를 가진다.

### 3.1 Stop the World & Mark the Sweep

`Stop the world`는 이름처럼 GC를 실행하기 위해서 GC를 실행하는 쓰레드를 제외한 모든 쓰레드의 작업을 멈추는 것을 말한다. 그리고 쓰레드의 작업이 멈추면 어플리케이션도 멈추기 때문에 GC의 성능 개선을 위해 튜닝을 한다고 하면 보통 Stop the world의 시간을 줄이는 작업을 말한다.

`Mark the Sweep` 은 Stop the World 로 쓰레드의 작업을 중단시키면 이제 모든 변수와 Reachable 객체를 스캔하면서 각각 어떤 객체를 참조하고 있는지 확인한다. 그리고 사용되지 않는 메모리를 구별해 (= Mark), 메모리에서 제거한다(= Sweep).

- **Mark** : 사용되는 메모리와 그렇지 않는 메모리를 구별하는 작업. 어플리케이션이 일시 중지되면 GC는 참조되고 있는 객체와 연결된 객체를 타고 이동하며 접근 가능한 객체를 식별한다.
- **Sweep** :사용되지 않는 메모리를 해제하는 작업

![다운로드](https://github.com/princenim/TIL/assets/59499600/15e59cf9-b7f0-4346-a475-f7b152546e73)

### 3.2 Minor GC


  <img width="739" alt="gcgc" src="https://github.com/princenim/TIL/assets/59499600/ed501afb-5c45-40e4-b25a-d7758871b9b2">

먼저 이 Minor GC는 young 영역에서 실행되는 GC이다. Young 영역은 1개의 Eden 영역, 2개의 Survivor영역으로 총 3개의 영역으로 나눠진다.

- **Eden :** new 를 통해 새로 생성된 객체가 위치한다. 정기적인 GC후 살아남은 객체는 Survivor영역으로 이동한다.
- **Survivor 0 / Survivor 1 :** 최소 1번 이상 살아남은 객체가 존재하는 영역이다.

1. 객체가 처음 생성되면 `eden` 영역으로 객체가 생성된다.
2.  `Eden` 영역이 꽉 차면 Minor GC가 실행한다. Mark 동작을 통해 reachable 객체를 탐색한다. 이때 살아남은 객체를 Survivor1에 이동한다. 이때 더 이상 사용되지 않는 메모리는 해제된다.(Sweep)
3. 살아남은 모든 객체들은 age 값 1이 증가한다.
4. 또 다시 `Eden` 영역에 신규 객체들이 꽉차면 Minor GC가 발생한다.
5. 다시 Eden  영역이 꽉찼을 때 Minor GC가 발생하며 살아남은 객체와 기존 `Survivor1`에 있던 객체들을 `Survivor2`로 이동시키고, Survivor1과 Eden 영역을 초기화시킨다.
6. `Survivor1` 영역이 꽉 차면 살아남은 객체는 `Survivor2` 영역으로 이동한다. `Survivor`둘 중 한공간은 반드시 빈 상태가 되야한다. 이때 두 `Survivor` 영역 중 1개는 항상 사용량이 0을 유지해야한다.
7. 그리고 이 과정을 반복해 임계값이 다다른  객체는 `Old 영역`으로 이동한다. 이를 **Promotio**n이라고 한다.

여기서 **age 값**이란 `Survivor` 영역에서 객체가 살아남은 횟수를 의미하며 Object Header에 기록된다. 만약 이 age값이 임계값이 다다르면 Old 영역으로 이동한다. 기본 임계값은 31이다.

### 3.3 Major GC

major GC는 old 영역이 꽉차 더 메모리가 부족해지면 발생한다. Young 영역은 일반적으로 Old 영역보다 크기자 작아서 GC가 보통 0.5초에서 1초 사이에 끝난다. 그래서 자바 어플리케이션에 크게 영향을 주지 않는다. 하지만 Old영역에는 Young 영역 보다 크기 때문에 Minor GC보다 시간이 오래 걸린다.

즉 , **minor GC가 발생하는 시점은 Young 영역의 Eden 영역이 꽉찼을때, major GC가 발생하는 시점은 old 영역이 꽉찼을 때이다.**

## 5. 왜 Young과 Old로 영역을 나눴을까?

이 영역을 나눈 이유는 위에서 말한 가설에서 찾을 수 있다.

‘**Weak Generational hypothesis’**

- **대부분의 객체는 금방 접근 불가능한 상태(Unreachable)가 된다.**
- **오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.**

따라서 두 영역을 나누어 놓고 힙 전체에서 GC를 수행하지않고 먼저 Young 영역을 수행해서 대부분의 가비지가 수거되기 때문에 메모리 낭비를 막을 수 있고, 더 효율적으로 GC를 수행할 수 있기 때문이다.

## 4.다양한 GC 알고리즘

## 5. GC의 단점

이러한 개발자에게 편리한 GC에도 단점이 존재한다.

1. 개발자가 메모리가 언제 해재되는지 정확하게 알 수 없다.
2. GC가 동작하는 동안에는 다른 동작을 멈춰서 오버헤드가 발생할수있으며 이는 어플리케이션의 성능저하의 원인이 될 수 있다.
    1. **오버헤드란?** 특정 기능을 수행하는데 간접적으로 드는 시간과 메모리 등을 말한다. 예를들어 A라는 기능을 처리하는데 단순히 10초가 걸리는데 20초가 걸렸다면 오버헤드는 10초인것이다.