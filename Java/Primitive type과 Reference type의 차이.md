# 1. Primitive type 과 reference type 의 차이 

자바에는 크게 Primitive type 과 reference type가 있다.

## 1.1 기본 자료형(primitive type)

- Primitive type은 기본 자료형으로 정수, 실수, 문자 ,boolean 등의 **실제 데이터 값을 저장한다.**
- e.g. byte, short,int, long
- JVM의 Runtime data area의 **Stack 영역에 저장한다.**
- 객체가 아니기때문에 nul l값을 가질 수 없다.  따라서 기본형에 null을 넣고 싶다면 `Wrapper` 클래스를 이용해야한다.

### 기본 자료형을 왜 stack에 저장할까?

> `because they have a fixed size and their lifetime is determined by the scope in which they are declared.`
>

이유는 기본 자료형은 크기가 고정되어있고, 선언되는 범위(= 수명 =메소드 범위)에 따라 주기가 결정되기 때문이다.
즉! 메소드 안에서만 사용되고, 참조가 불가능 하기때문이다.

먼저 `Runtime data area`의 `stack` 영역에는 ‘기본 자료형’에 해당하는 **지역변수 , 매개변수,리턴값이** 저장된다. 지역변수는 메소드안에 정의된 변수를 말한다. 즉, 메소드 안의 변수만이 `stack`에 저장된다.

따라서 메소드가 호출되면 `스택` 영역에 `스택 프레임`이 생성되고, 그 안에서 메소드를 호출한 후  메소드의 ‘기본자료형’에 해당하는 매개변수와 리턴값, 지역변수가 저장된다. 그리고 **이 데이터 값들은 무조건 메소드 안에서만 사용되고 다른 영역에서 참조하거나 크기가 바뀔일이 없다.** 따라서 메소드를 호출했을 때 메모리에 할당되며, 메소드 호출 범위가 종료되면 메소드와 같이 스택에서 할당 해제된다.

## 1.2 참조 자료형(reference type)

- reference type는 객체의 주소를 참조하는 타입으로 **참조변수는 Stack에, 객체의 주소값은 Heap에 저장해 참조변수가 객체의 주소를 참조하는 타입이다.**
- e.g. String, class, Interface ,Array
- 기본자료형을 제외한 모든 타입이 reference type이다.
- 빈 객체를 저장하는 null 값을 저장할 수 있다.

## 1.3 Primitive type 과 reference type의 차이

1. **실제 데이터 값 저장 위치**

**기본형은 데이터가 Stack에  저장된다.**
**참조자료형은 실제 데이터값은 Heap에 변수는 Stack에 저장되어 변수 공간에 Heap에 값이 저장된 메모리 주소값을 저장한다.**

2. **null을 담을 수 있는가**

기본 자료형은 null을 담을 수 없고, 참조 자료형은 null을 담을 수 있다.

```java
int i = null; //불가능
Integer integer = null // 가능
```

3. 제네릭 타입에서 사용 여부

기본 타입은 제네릭 타입에서 사용할 수 없지만, 참조 자료형은 가능하다 .

```java
List<i> list;// 불가능
List<Integer> list; // 가능
```

## 1.4 Wrapper 클래스 - int와 Integer의 차이

프로그램에 따라서 기본 타입의 데이터를 객체로 취급해야하는 경우가 있다.

예를들어 메소드의 매개변수가 객체 타입만이 요구될때는 기본타입을 참조자료 형으로 바꿔야 한다.  이때 사용할수있는게   `Wrapper` 클래스이다.

이렇게 **8개의 기본 타입에 해당하는 데이터를 객체로 포장해주는 클래스를  `Wrapper` 클래스라고한다.**

이  `Wrapper` 클래스는 모두 `java.lang` 패키지에 포함되어 있다.

```java
Integer a = new Integer(10);
Integer b = new Integer("1000");

Long a = new Long(10);
Long b = new Long("1000");

Character a = new Character('a');
```

| 기본타입 | 래퍼 클래스  |
| --- | --- |
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| char | Character |
| boolean | Boolean |

### 박싱(Boxing)과 언박싱(UnBoxing)

![wrapper](https://github.com/princenim/TIL/assets/59499600/1e7d28e5-7a36-4951-a438-acdcac8ec436)


`Wrapper` 클래스는 산술 연산을 위해 정의된 클래스가 아니므로 인스턴스에 저장된 값을 변경할 수 없다

단지 값을 참조하기 위해 새로운 인스턴스를 생성하고, 생성된 인스턴스의 값만을 참조할 수 있다.

- **박싱(Boxing)** : 기본타입의 데이터를 `Wrapper` 클래스의 인스턴스로 변환
- **언박싱(UnBoxing)** : `Wrapper` 클래스의 인스턴스에 저장된 값을 다시 기본 타입의 데이터로 꺼내는 과정

### Wrapper 클래스가 필요한 이유

1. 매개변수로 객체가 요구될때
2. 기본값이 아닌 객체로 저장해야할때
3. 객체간의 비교가 필요할때