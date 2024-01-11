# 📌 GC(Garbage Collector)의 개념 및 동작원리

# 1. GC (Garbage Collection)

**GC(Garbage Collector, Garbage Collection)는 JVM의 힙 영역에서 더 이상 참조되지않는 객체를 메모리에서 해제시켜주는 장치이다.** JVM 내에서 자동으로 동작하기 때문에 Java는 개발자가 특별한 경우가 아니면 메모리 관리를 개발자가 직접해주지 않아도 된다.

![gc](https://github.com/princenim/TIL/assets/59499600/07761a59-bf26-4be1-be52-dca536c38d7f)

```java
Person person = new Person();  //참조변수 person이 객체를 가리킴
person.setName("hong")
person = null; //참조변수 person이 null로 더 이상 객체를 참조하지 않음.
```

위의 코드에서 person 객체는 더 이상 참조받지않고, 사용되지 않아서 Garbage가 되었다. 따라서 자바는 이러한 메모리 누수를 방지하기 위해 Garbage Colloctor를 이용해 주기적으로 메모리를 청소한다.

# 2. Young 영역과 Old 영역

JVM의 Heap영역은 처음 설계될 때 2가지 조건으로 설계되었다.

‘**Weak Generational hypothesis’**

- **대부분의 객체는 금방 접근 불가능한 상태(Unreachable)가 된다.**
- **오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.**

다시 말하면 객체는 대부분 일회성이며 보통 새로운 객체에서 오래된 객체를 참조한다는 것이다. 이러한 가정에서 Heap영역을 물리적으로 2개로 나누었는데 그게 Young 영역과 , Old 영역이다.

![heap ](https://github.com/princenim/TIL/assets/59499600/28b664d5-712f-4242-8fd3-2441925b32f8)

## 2.1 Young Generation(Young 영역)

새로운 객체가 할당되는 영역을 말한다. 대부분의 객체가 금방 *Unreachable*한 상태가 되서 생성되고 금방 사라진다. 이 영역에 대한 GC를 **Minor GC**라고 한다.

## 2.2 Old Generation(Old 영역)

Young 영역에서 reachable한 상태를 유지해 살아남은 객체가 복사된 영역이다.
복사되는 과정에서 대부분 Young 영역보다 크게 할당된다. 이 old 영역에 대한 GC를 **Major GC라**고 한다.

![car table](https://github.com/princenim/TIL/assets/59499600/e693f4aa-8f21-44ae-87ac-765fa775bb24)

그렇다면 예외적으로 Old 영역에서 Young 영역의 객체를 참조하는 경우는 어떻게 해야할까? 이러한 경우를 대비해 **Old 영역에는 512bytes의 덩어리로 되어있는 카드 테이블이 있다.** 카드 테이블은 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 관련된 정보가 생긴다. 따라서 Minor GC를 실행할때 Old 영역에 있는 모든 객체의 참조를 확인하지 않고, 이 카드 테이블만 확인해서 GC대상인지 식별한다.

### 왜 Young과 Old로 영역을 나눴을까?

이 영역을 나눈 이유는 위에서 말한 가설에서 찾을 수 있다.

‘**Weak Generational hypothesis’**
- 대부분의 객체는 금방 접근 불가능한 상태(*Unreachable)* 가 된다.
- 오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.

따라서 두 영역을 나누어 놓고 힙 전체에서 GC를 수행하지않고 먼저 Young 영역을 수행해서 대부분의 가비지가 수거되기 때문에 메모리 낭비를 막을 수 있고, 더 효율적으로 GC를 수행할 수 있기 때문이다.

# 3. GC의 동작원리

Young 영역과 Old 영역은 위의 그림으로 보듯이 서로 다른 메모리 구조를 가지고 있다. 세부적인 동작은 다르지만 하지만 기본적으로 다음과 같은 GC의 공통적인 단계를 가진다.

## 3.1 Stop the World & Mark the Sweep

`Stop the world`는 이름처럼 GC를 실행하기 위해서 GC를 실행하는 쓰레드를 제외한 모든 쓰레드의 작업을 멈추는 것을 말한다. 그리고 쓰레드의 작업이 멈추면 어플리케이션도 멈추기 때문에 GC의 성능 개선을 위해 튜닝을 한다고 하면 보통 Stop the world의 시간을 줄이는 작업을 말한다.

`Mark the Sweep` 은 Stop the World 로 쓰레드의 작업을 중단시키면 이제 모든 변수와 Reachable 객체를 스캔하면서 각각 어떤 객체를 참조하고 있는지 확인한다. 그리고 사용되지 않는 메모리를 구별해 (= Mark), 메모리에서 제거한다(= Sweep).

- **Mark** : 사용되는 메모리와 그렇지 않는 메모리를 구별하는 작업. 어플리케이션이 일시 중지되면 GC는 참조되고 있는 객체와 연결된 객체를 타고 이동하며 접근 가능한 객체를 식별한다.
- **Sweep** :사용되지 않는 메모리를 해제하는 작업

![다운로드](https://github.com/princenim/TIL/assets/59499600/15e59cf9-b7f0-4346-a475-f7b152546e73)

###  Stop the world 시 왜 쓰레드를 멈춰야할까?

그렇다면 드는 의문이 gc를 할때 어플리케이션이 멈추면 손해가 발생할텐데 왜 쓰레드를 멈출까?

이는 gc에서 메모리를 계속 읽을때 한번에 많은 메모리를 읽다가 중간에 객체가 바뀌면 처음부터 다시 읽어야하니까 아예 쓰레드를 멈추고 gc를 실행한다. 즉, 어플리케이션 쓰레드가 멈춰야 현재 메모리 상에서 객체를 정확히 식별할 수 있기때문이다.


## 3.2 Minor GC


  <img width="739" alt="gcgc" src="https://github.com/princenim/TIL/assets/59499600/ed501afb-5c45-40e4-b25a-d7758871b9b2">

먼저 이 Minor GC는 young 영역에서 실행되는 GC이다. Young 영역은 1개의 Eden 영역, 2개의 Survivor영역으로 총 3개의 영역으로 나눠진다.

- **Eden :** new 를 통해 새로 생성된 객체가 위치한다. 정기적인 GC후 살아남은 객체는 Survivor영역으로 이동한다.
- **Survivor 0 / Survivor 1 :** 최소 1번 이상 살아남은 객체가 존재하는 영역이다.

1. 객체가 처음 생성되면 `eden` 영역으로 객체가 생성된다.
2.  `Eden` 영역이 꽉 차면 Minor GC가 실행한다. Mark 동작을 통해 reachable 객체를 탐색한다. 이때 살아남은 객체를 Survivor1에 이동한다. 이때 더 이상 사용되지 않는 메모리는 해제된다.(Sweep)
3. 살아남은 모든 객체들은 age 값 1이 증가한다.
4. 또 다시 `Eden` 영역에 신규 객체들이 꽉차면 Minor GC가 발생한다.
5. 다시 `Eden` 영역이 꽉찼을 때 Minor GC가 발생하며 살아남은 객체와 기존 `Survivor1`에 있던 객체들을 `Survivor2`로 이동시키고, `Survivor1`과 `Eden` 영역을 초기화시킨다.
6. `Survivor1` 영역이 꽉 차면 살아남은 객체는 `Survivor2` 영역으로 이동한다. `Survivor`둘 중 한공간은 반드시 빈 상태가 되야한다. 이때 두 `Survivor` 영역 중 1개는 항상 사용량이 0을 유지해야한다.
7. 그리고 이 과정을 반복해 임계값이 다다른  객체는 `Old 영역`으로 이동한다. 이를 **Promotio**n이라고 한다.

여기서 **age 값**이란 `Survivor` 영역에서 객체가 살아남은 횟수를 의미하며 Object Header에 기록된다. 만약 이 age값이 임계값이 다다르면 Old 영역으로 이동한다. 기본 임계값은 31이다.

### **gc 과정에서 왜 survivor 한곳을 비워야할까?**

> `You must be wondering why do we have 2 survivor space. We have 2 survivor spaces to avoid memory fragmentation. Each time you copy objects from eden to survivor and you get empty eden space and 1 empty survivor space.`
>

gc가 일어나 `eden`이나 `survivor` 영역에 객체가 제거되면 파편화된 메모리 영역이 존재하게 되는데 이를 한 영역에 모아놔서 **순차읽기/쓰기**를 하려고 한곳을 비운다.
이는 ‘디스크 조각 모음’과 비슷하다고 보면 된다.  하드 디스크에 파일 저장시에 이곳 저곳에 흩어져서 저장되는 것을 **단편화(fragmentation)** 라고 하는데 그리고 이 단편화가 발생한 디스크들을 한 덩어리로 모아 재배치해주는게 **디스크 조각 모음** 이다.  이렇게 디스크 조각 모음을 하게 되면 순차읽기/쓰기를 하게 되어 속도가 빨라지게 된다.

따라서 gc가 일어날때 `survivor`의 한 곳에 모으는 이유도 더 빠르게 성능을 위함이며, 이러한 점이 결국 메모리 관리를 하지 않아도 되는 자바의 장점이기도 하다.

(추가로 보통 하드디스크에서 ‘삭제’라는 작업은 매우 비용이 높은 작업이다. 따라서 연속된 메모리에서 중간에 값이 데이터상으로 삭제되었을 때 실제 메모리의 값을 삭제하지않고, 다른곳에서 이곳이 사용되지 않고있다고 체크하고 다음에 사용할때 체크한 곳을  보통 덮어씌우는 방식을 사용한다. )

## 3.3 Major GC

major GC는 old 영역이 꽉차 더 메모리가 부족해지면 발생한다. Young 영역은 일반적으로 Old 영역보다 크기자 작아서 GC가 보통 0.5초에서 1초 사이에 끝난다. 그래서 자바 어플리케이션에 크게 영향을 주지 않는다. 하지만 Old영역에는 Young 영역 보다 크기 때문에 Minor GC보다 시간이 오래 걸린다.

즉 , **minor GC가 발생하는 시점은 Young 영역의 Eden 영역이 꽉찼을때, major GC가 발생하는 시점은 old 영역이 꽉찼을 때이다.**

# 4. GC의 단점

이러한 개발자에게 편리한 GC에도 단점이 존재한다.

1. 개발자가 메모리가 언제 해지되는지 정확하게 알 수 없다.
2. Stop the world 즉, GC가 동작하는 동안에는 다른 동작을 멈춰서 오버헤드가 발생할수있으며 이는 어플리케이션의 성능저하의 원인이 될 수 있다.
   1. **오버헤드란?** 특정 기능을 수행하는데 간접적으로 드는 시간과 메모리 등을 말한다. 예를들어 A라는 기능을 처리하는데 단순히 10초가 걸리는데 20초가 걸렸다면 오버헤드는 10초인것이다.

## 5. GC 알고리즘

위에서 언급한 단점을 해결하기 위해 GC에는 다양한 알고리즘 방법이 존재한다.

## 5.1 Serial GC

이 GC 알고리즘은 Young 영역과 Old 영역이 다르게 실행된다.  Young의 경우 위에서 설명한 Mark and Sweep방식을 따르며 Old 영역의 경우 Compact를 추가한 **Mark Sweep Compact**가 사용된다.
즉, 객체를 식별하고(Mark) , Mark후 살아남지 못한 객체들을 메모리에서 제거 (Sweep) → Sweep 후 마지막까지 분산되어 살아있는 객체들을 앞으로 모아준다.(Compact)
이 Serrial GC의 특징으로는 싱글쓰레드로 GC를 처리하는 쓰레드가 한 개이다. 따라서 느리고 Stop the World 시간이 다른 GC에 비해 길다. 따라서 메모리나 CPU 코어 리소스가 부족할 때 사용한다.

## 5.2 parallel GC

Serial GC를 멀티쓰레드로 실행하는 GC이다. **throughout collector**로도 알려진 방법이며 Java 8의 default GC이다. Serial GC와 기본적인 과정이 동일하나 Young 영역의 GC를 병렬로 수행한다. Old영역은 Serial GC와 마찬가지로  Mark Sweep Compact를 사용된다. 따라서 **Parallel GC는  young 영역의 GC를 멀티쓰레드로 수행하기때문에 속도가 빠르다.**

<img width="1005" alt="p gc" src="https://github.com/princenim/TIL/assets/59499600/9a36876b-3679-442d-8318-0c06ef464823">


위의 그림을 보면 Serial GC와 parallel GC의 Stop the world 부분에서 쓰레드의 차이를 확인할 수 있다.

## 5.3 parallel Old GC

parallel Old GC는 parallel GC의 Old GC를 개선해 멀티쓰레드로 업그레이드 한 버전이다. Old 영역에서 **Mark Summary Compact** 알고리즘을 사용하는게 차이점이다. Mark Summary Compact에서 Sweep의 경우 단일 쓰레드가 Old 영역을 찾아 살아있는 객체만 찾아내는 방식인데 반해, Summary는 여러 쓰레드가 Old 영역을 나눠 병렬적으로 동시에 찾아낸다.

## 5.4 CMS GC(Concurrent Mark Sweep GC)

![cms gc](https://github.com/princenim/TIL/assets/59499600/15425b37-1209-49c0-89f2-45a5a1f77158)

CMS GC는 GC 과정을 어플리케이션 쓰레드와 동시에(Concurrently) 수행함으로 Stop the World 시간을 최소화하기위해서 고안된 방식이다.  Compact 과정이 사라졌으며 다음과 같은 단계를 거친다.

1. **Initial Mark (STW 발생)** : Old 영역에 있는 객체중에 GC 루트 객체에서 직접 참조하는 객체 또는 Young 영역의 객체에서 참조하는 객체만 **빠르게** 탐색해 marking 함으로써 Stop the World시간을 최소화한다. .
2. **Concurrent Mark** : Initial Mark 단계에서 마킹된 객체가 참조하는 객체를 대상으로 살아있는 객체를 추가 확인한다. 이때 어플리케이션 작업과 동시에 수행하므로 Stop the World가 발생하지 않는다**.**
3. **Remark(STW 발생)** : Concurrent Mark 과정에서 변경된 사항이 없는지 다시 한 번 마킹하며 확인한다.
4. **Concurrent Sweep** :  접근할수 없는 객체를 제거한다.

여기서 **GC root란**  가비지 컬렉션에는 GC root가 있는데 이는 힙 외부에서 접근할 수 있는 변수나 오브젝트를 말한다. 즉 가비지 컬렉션의 Root라는 의미이다.

## 5.5 G1(Garbage First GC)

![g1](https://github.com/princenim/TIL/assets/59499600/0188ee15-a439-4d2d-ad7b-11aae76be787)

CMS GC를 개선한 방식으로 Java9 이상의 디폴트 GC이다. 현재 GC중 Stop the World의 시간이 가장 짧다.

G1 GC는 전통적으로 Heap을 `Old`와 `Young`으로 구분하던 방법과는 달리 물리적으로 영역을 구분하지 않는다. 대신 Heap 을 균등한 크기의 **region(영역)** 으로 구분해 논리적으로 구분한다 . 그리고 `eden` , `survivor`, `old`에  `Humonogous`와 `Availabe/Unused`를 역할을 추가했다. `Humonogous`는 region 영역 크기의 50%가 넘은 객체를 저장하는 region, `Availabe/Unused`은 사용되지 않는 region을 의미한다.

동작방식은 다음과 같다.

- **Minor GC** : 한 지역에 객체를 할당하다가 해당 지역이 꽉 차면 Minor GC가 실행된다. G1 GC는 각 지역을 추적하고 있기 때문에 가비지가 가장 많은 지역을 찾아 Mark And Sweep을 한다. 만약 Eden 지역에서 GC가 수행되면 Mark and Sweep을 하고 살아남은 객체를 다른 지역으로 이동시킨다. 이 지역이 `Availabe/Unused` 면 해당 지역은 `Survivor` 영역이 되고 , `Eden` 지역은 `Availabe/Unused` 이 된다.
- **Major GC:** 시스템이 계속 운영되다가 객체가 너무 많아 메모리를 회수 할 수 없을 때 Major GC가 발생한다여기서 가장 큰 차이점이 발생하는데 기존의 다른 GC는 모든 Heap 의 `Old` 영역에서 GC가 수행하는데 그에 따라 시간이 오래걸렸다. 하지만 G1 GC는 어디 영역에 가비지가 가장 많은지 알고있기 때문에 해당 영역만 GC를 수행한다.

따라서 G1 GC는 메모리를 일일히 탐색해 객체를 제거하지않고, 가비지가 많은 영역을 먼저 GC하기 때문에 속소가 빠르다. 그리고 이전 GC들은 `Eden` → `Surivivor0` → `Survivor1` 로 순차적으로 이동했지만 G1 GC는 순차적으로 이동하지 않는다. 대신 효율적이라고 생각하는 위치로 객체를 이동시킨다.