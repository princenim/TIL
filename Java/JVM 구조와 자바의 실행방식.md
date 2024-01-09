# JVM 구조와 자바의 실행방식

# 1. JVM이란

JVM이란 **Java Virtual Machine**으로 자바를 실행하기 위한 가상 컴퓨터이다. 자바와 운영체제 사이에서 중개자 역할을 하며 **자바가 운영체제에 구애받지않고 프로그램을 실행할 수도록 도와주는 역할**을 한다.

더 정확히 말하면 자바 바이트 코드를 실행시키기 위한 가상의 기계이다.

![jvm](https://github.com/princenim/TIL/assets/59499600/11c31447-c7a9-417d-8340-602a7e5f4f8c)

일반 어플리케이션은 OS(운영체제)에서를 거쳐 하드웨어에 전달되는데, 자바 어플리케이션은 JVM에서 OS,하드웨어로 전달된다. 따라서 자바 어플리케이션은 한 단계가 더 있어서 속도가 느리다는 단점을 가지만, OS에서 독립적이기때문에 JVM이 깔린 어느 컴퓨터에서도 쉽게 프로그램을 읽을 수있는 장점이 있다.

![jvm2](https://github.com/princenim/TIL/assets/59499600/9560d237-ba78-4295-8dc1-7f454c71bd16)

먼저 JVM의 구성요소를 알아보기전에 자바가 컴파일 하는 방식을 알아보자.

# 2. 자바의 실행방식

먼저 사람이 만든 소스코드는 컴퓨터가 해석을 할수없으니 해석을 하는 작업을 컴파일이라고 한다.

![java code](https://github.com/princenim/TIL/assets/59499600/c159a3c0-1c52-46d7-ad50-0d32e9515085)

1. 개발자가 자바 코드(`.java`)를 작성한다.
2. 자바 컴파일러 (`javac`)가 자바 코드(`.java`)를 컴파일 하면 `.class` 확장자인 Java byte code가 생성되어 디스크에 저장된다. 이 자바 바이트 코드는 JVM만 있으면 어느 운영체제에서도라도 실행된다.
3. 이때 JVM이 컴파일을 마친 `.class` 파일을 읽어서 운영체제에서 실행시킨다. 더 정확히 말하면 JVM의 `Class Loader`가 동적 로딩을 통해 필요한 클래스들을 로딩 및 링크하여 `Runtime Data Area`에 적재한다. 그리고 `Runtime Data Area`에 로딩된 바이트 코드는 `Excution Engine`을 통해 실행된다.

```java
public class ArrayMain {
    public static void main(String[] args) {
        System.out.println("success");
    }
}
```

```java
javac b/array/ArrayMain.java //javac이라는 컴파일러로 컴파일하면 .java형태의 파일이 생긴다. 
java b/array/ArrayMain //그 파일을 실행 
```

# 3. JVM 구성요소

![jvm architecture](https://github.com/princenim/TIL/assets/59499600/0012a66e-174d-45ee-8e9f-c4bb2e45f33f)

JVM은 크게 아래과 같이 구성되어 있다.

- **Class Loader**
- **Execution Engine(실행 엔진)**
    - **인터프리터 (interpreter)**
    - **JIT 컴파일러(Just-In-Time)**
    - **Garbage Collector(가비지 콜렉터)**
- **Runtime Data Area**
    - **PC register**
    - **Stack Area**
    - **Native Method Stack**
    - **Heap Area**
    - **Method Area**

## 3.1 Class Loader

![class loader](https://github.com/princenim/TIL/assets/59499600/7de43bec-ac1f-40cf-804b-8a75d1412d0a)

클래스 로더는 JVM 내로 클래스 파일`(.class)`을 로드하고, Link 작업을 통해 Runtime Data Area 적재하는 역할을 한다. 자바의 클래스들은 한 번에 모든 클래스가 메모리에 올라가지않으며, 필요할때 메모리에 올라간다.

클래스 로드는 **메모리를 로드하는 로딩**, **검증하는 링크** , **적절한 값으로 초기화나는 초기화** 3단계로 이루어져 있다.

### 1. **로딩(loading)**

컴파일된 `.class`의 자바 바이트 코드를 Runtime Data Area의 Method Area에 저장한다. 클래스, 인터페이스 , ENUM, 변수, 메소드 정보들을  저장한다. 로딩이 끝나면 해당 클래스 타입의 Class 객체를 생성해 Heap 영역에 저장한다.  한번에 메모리에 모두 로드하는 것이 아니라 필요한 경우 동적으로 메모리에 로드한다.

```java
public class HI {
   public static void main(String[] args) {
   }
}

class Check{

}
```

이와 같이 HI라는 `.class` 파일에 check 라는 클래스라고 존재해도 사용하지 않는다면 로드하지 않는다.

```java
public class HI {
    public static void main(String[] args) {
        int value = Check.value;
    }
}

class  Check{
    static int value = 10;
    
}
```

반면에 `Check`라는 클래스에 `static` 변수가 존재한다면 , `int value = Check.value` 라는 명령어를 만날 때 Check 클래스도 함께 로드된다.

하지만 해당 변수가 final 변수라면 상수가 되므로 Check 클래스는 로드되지 않는다. 그 이유는 상수는 Runtime Data Area에 Constant Pool에서 따로 저장되어 관리되기 때문이다.

### 3. **링크 (linking)**

읽어들인 클래스 데이터는 3단계의 링킹 과정을 거친다.

1. **verify** : `.class` 파일 형식이 유효한지 JVM 명세에 명시된 대로 잘 구성되어있는지 검사한다. 유효하지 않은 경우 런타입에러(java.lang.VerifyError)가 발생한다.
2. **prepare** : 클래스 변수(static 변수)와 기본값에 필요한 메모리를 준비하는 과정이다.
3. **resolution** : 선택적 단계로 심볼릭 메모리 레퍼런스를 메모리 영역에 존재하는 실제 레퍼런스로 교체한다.
    1. 심볼릭 메모리 레퍼런스란 ? 클래스의 특정 메모리 주소를 참조관계로 구성한 것이 아니라 참조하는 대상의 이름만을 지칭한 것을 말한다. 즉 실제 메모리 주소가 아니라 이름만을 가진다.

### 3. **초기화 (Initialization)**

링크에서 prepare단계에서 확보한 메모리 영역에 클래스 변수들 (= static 변수)를 적절한 값으로 초기화한다. 즉, static 필드들이 설정된 값으로 초기화한다.

## 3.2 **Runtime Data Area**

![run-time](https://github.com/princenim/TIL/assets/59499600/e2f373e1-6b64-464d-87ac-2a5214942f53)

![run](https://github.com/princenim/TIL/assets/59499600/2b44199a-9bbe-495d-bc65-0c63ec126613)

**OS으로부터 할당받은 JVM의 메모리 영역**으로 자바 어플리케이션을 실행할 때 사용되는 데이터를 적재하는 영역이다. 5가지영역으로 나누어져 있다.

### 1. PC register

PC register은 Thread가 생성될 때마다 생기는 공간으로 현재 ‘**실행 중인 JVM 명령어 주소값**’를 저장하는 공간이다. 즉 코드의 몇번째 줄을 실행하면 되는지 저장하는 공간이다. 따라서 쓰레드가 시작될 때 초기화되며 쓰레드가 종료되면 메모리에서 해제된다. 각 쓰레드는 자신의 PC 레지스터를 하나씩 가지고 있다. 따라서 다중 쓰레드 환경에서 동시에 여러 명령어를 실행할 수 있다.

일반적으로 프로그램의 실행은 CPU에서 명령어를 수행하는 과정으로 이루어진다. CPU는 컴퓨터의 연산을 실행하고 제어하는 가장 중요한 일을 맡고 있다. 이 CPU가 연산을 수행하는 동안 필요한 정보를 CPU 내의 기억장치를 이용해 기억하는데 이를 PC register라고 한다.

하지만 자바의 PC register은 다르다. 자바는 OS 입장에서 단순히 하나의 프로세스이기때문에 JVM을 사용해야한다. 그래서 자바는 CPU에 직접 연산을 수행하도록 하는 것이 아닌 현재 작업하는 내용을 CPU에게 연산으로 제공해야하며, 이때 필요한 공간으로 PC register 라는 메모리 영역을 만든 것이다. 따라서 JVM은 스택프레임에 있는 Operand라는 스택 영역에서 명령어를 뽑아서  PC regtster에 저장한 후 CPU에게 명령을 전달한다.

### 2.  Stack Area

Thread가 생성될 때마다 생기는 공간이며 **int, long, boolean 등 기본 자료형(primitive type)을 생성할 때 저장하는 공간**이다. 임시적으로 사용되는 변수나 정보들이 저장되는 영역을 말한다. 뿐만 아니라 **Heap 영역에 생성된 객체들을 참조하는 주소값이 할당된다.**

Stack 자료구조는 Last in First Out 구조로 마지막에 들어온 값이 가장 먼저 나간다. 메소드 호출시마다 각각의 스택 프레임이 생성되고 메소드 안에서 사용되는 값들을 저장하며 호출된 메소드의 매개변수, 지역변수, 리턴값을 다 임시로 저장한다. 메소드가 호출될떄 메모리에 할당, 종료되면 메모리에서 사라진다.

### 3. Native Method Stack

![native](https://github.com/princenim/TIL/assets/59499600/c3e08855-4d8d-4f32-868e-3bb626344045)

Java로 작성된 프로그램을 실행하면서 순수하게 Java로 구성된 코드만을 사용할수 없는 시스템의 자원이나 API 가 존재한다. 다른 언어로 작성된 메소드들을 Native Method 그리고 이 메소드들을 다루는 영역이 Native Method Stack이다.

즉, **자바 이외의 언어 (C, C++, 어셈블리)로 작성된 메소드를 지원하기 위한 스택**이며 이를 호출할 때 JVM Stack은 동작하지 않고, Native Method Stack을 활용해 push, pop한다. 쓰레드별로 1개씩 존재한다.

### 4. Heap Area

![heap](https://github.com/princenim/TIL/assets/59499600/ee6a8f28-8843-4515-a5ec-acff5abce649)

자바 프로그램에서 사용되는 모든 객체들이 저장되는 영역이다 `**new` 를 사용하여 객체를 생성하면 Heap 영역에 저장된다.**  힙의 참조주소는 Stack Area가 갖고 있고 해당 참조변수를 통해서만 힙 영역에 존재하는 인스턴스를  핸들링할 수 있다.

**모든 쓰레드가 함께 공유하며** JVM이 관리하는 프로그램상에서 데이터를 저장하기 위해 런타임시 동적으로 할당해 사용하는 영역이다. 따라서 만약이 Heap 영역에 있는 인스턴스를 참조하는 변수나 필드가 없다면 의미 없는 객체가되어 JVM이 필요없다고 판단, GC의 대상이 되어 자동으로 삭제된다.

### 5. Method Area (= Static Area = Class Area)

**Method Area에는 클래스 로더가 적재한 클래스 혹은 인터페이스에 대한 메타 데이터 정보가 저장된다.** JVM이 실행될때 method area는 생성되며, (java 명령어로 실행할때 main 메소드 호출하며 실행된다.) 클래스가 용되면 JCM은 해당 클래스의 .class 파일을 읽어 분석해 클래스의 인스턴스 변수, 메소드 정보 등을 method area에  적재한다. 이 때 클래스 변수도 이 영역에 함께 생성된다. 즉 클래스가 로드 되는 시점은 해당 클래스가 사용되기 위해 호출되는 때이다.

참고로 프로그램이 실행되었다고 모든 코드가 저장되어 있는 상태가 아니다. new 객체가 생성되기 이전에는 텍스트일 뿐이다. 따라서 객체 생성 후에 메소드를 실행하게 되면 해당 클래스 코드에 대한 정보를 Method Area에 저장하게 된다.

여기서 메타 데이터는 다음과 같다.

- **Runtime Constant Pool**:  메소드 영역에 존재하는 별도의 관리 영역을 말한다. 상수 자료형을 저장하며 중복을 막는 역할을 한다. 인덱스로 접근이 가능하다.
- Field Information (멤버 변수) : 멤버 변수의 이름, 데이터 타입, 접근 제어자
- Method Information (메소드) : 메소드 이름, return 타입, 매개변수 , 접근 제어자
- Type Information (타입): 클래스와 인터페이스를 말한다.  이름, 하위 클래스 이름, 클래스와 인터페이스 여부 , 제어자
- Class Variable
    - staic 키워드로 선언된 변수
    - 모든 인스턴스에 공유되어 인스턴스 없어도 직접 접근이 가능하다.
    - 클래스를 사용하기 이전에 미리 메모리를 할당 받아 있는 상태가 된다.

## 3.3 Execution Engine (실행엔진)

![execetion engine](https://github.com/princenim/TIL/assets/59499600/67c544d6-6553-45ba-b559-cd8b5895f681)

Execution Engine 이란, 클래스로더를 통해 Runtime Data Area에 적재된 자바 바이트 코드들을 명령어 단위로 읽어서 실행하는 역할을 한다. 즉 메모리에 할당된 바이트 코드를 실행하는 역할을 담당한다.

이 실행엔진에는 인터프리터, JIT Complier, Garbage Collections이 있다.

### 1. 인터프리터(Interpreter)

javac 이라는 자바 컴파일러로 변환된 자바 바이트 코드를 컴퓨터가 이해할 수 있도록 Native Code로 바꾸는 작업을 한다. **한줄 한줄 순차적으로** 컴파일을 하여 Native로 변환하는 작업을 하는데 이는 매번 컴파일을 할때 비효율적이며 중복된 바이트 코드에 대해서는 JIT 컴파일러를 사용한다.

### 2. JIT(Just In Time) Compiler

인터프리터의 단점을 개선하기 위해 등장한 방법으로 인터프리터가 반복되는 코드를 발견하면 JIT 컴파일러로 반복되는 코드를 모두 Native Code로 바꾼다. 그리고 그것을 캐싱을 하고 동일한 부분이 호출되면 캐싱을 한 코드를 불러 한 번더 인터프리터를 통해 컴파일하지 않도록 한다.

따라서 한번 바꾼 Native Code는 인터프리터가 더이상 컴파일하지 않아도 된다.

### 3. Garbage Collections (GC)

GC는 JVM 상에서 더 이상 참조되지않는 객체를 메모리에서 해제시켜주는 장치이다. JVM내에서 자동으로 동작하기 때문에 Java는 개발자가 특별한 경우가 아니면 메모리 관리를 개발자가 직접해주지않아도 된다.
더 자세한 설명은 [GC의 개념 및 동작원리](https://github.com/princenim/TIL/blob/master/Java/GC%EC%9D%98%20%EA%B0%9C%EB%85%90%20%EB%B0%8F%20%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC.md)에서 확인할 수 있다. 