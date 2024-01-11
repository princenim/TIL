# 1. String literal 과 String object의 차이

자바에는 스트링을 선언하는 방식이 리터럴과 객체(object) 2가지가 있다.

이 두 방식의 차이는 문법 뿐만 아니라 **실제 메모리에 할당되는 영역에도 차이가 있다.**

## 1.1 String object

`new` 키워드를 이용했기 때문에 다른 객체처럼 실제값은 `Heap`에 참조변수 `a`는 `Stack`에 저장된다. 그리고 변수 `a` 가 주소값을 참조한다.

```java
 String str1 = new String("Hello");
```

## 1.2 String literal

일반적으로 객체를 생성할 때는 `new` 키워드를 사용해서 생성한다. 하지만 문자열은 특이하게 키워드 없이 생성할 수있는데 이것을 **리터럴** 이라고 부른다.

```java
String str2 = "Hello"; 
String str3 = "Hello"; 
```

문자열 리터럴을 이용해 문자열을 생성하면 `Heap영역`의 `String Constant Pool`에 저장된다.

원래 Java 7이전에는 Perm 영역에 존재했으나 Java 7부터 Heap영역으로 옮겨졌다

## 1.3 메모리 상의 차이

![string](https://github.com/princenim/TIL/assets/59499600/a81ae533-89d4-4beb-876a-7e0650d6bbac)

위의 예시를 보면 리터럴로 생성한 str2,3은 `String constant pool`에 위치한 객체를 바라보고 , `new` 키워드로 생성한 str1은 다른 메모리 주소를 바라본다. 즉 여기서 알수있는 점이

1. String 리터럴은 JVM의 Runtime data area의 `Heap영역`의 `String constant pool` 에 저장된다.
2. String 리터럴로 생성한 객체가 이미 `String constant pool`에 존재하면 해당 객체는 이미 생성되어있는 주소값을 참조한다.
3. String 리터럴로 생성한 객체가 이미 `Heap`영역에 있어도 `new` 방식으로 객체를 생성한다면 별도의 객체가 생긴다.

**그렇다면 어떻게 String 리터럴로 생성한 객체가 같은 값이면 같은 객체를 바라보게 하는게  가능할까?**

`String contant pool`는 `Heap영역`에 위치해 있는데 `HashMap`으로 구현되어있다. 한 번 생성된 문자열 리터럴은 변경할 수 없으며 리터럴로 문자열을 생성하면 내부적으로 `String` 의 `intern()`이 호출된다. 이 `intern()` 메소드는  `String conatant pool`에 **같은 값이 있는지 찾고, 같은 값이 있으면 해당 주소값을 반환,  없다면 pool에 문자열을 등록 해당 주소값을 반환한다.**

```java
String  str1= "hello";
String  str2= "hello";

System.out.println(str1.intern()); //hi 
```

### Intern() 메소드

```java
//java.lang.String
public native String intern();
```

여기서 `native`란 자바 프로그램에서 다른 언어로 작성된 코드를 실행할 수 있는 JNI(Java Native Interface)키워드이다.


## 1.4  스트링 리터럴의 존재 이유

일단 둘다 불변 객체이다. 따라서 이 불변객체의 단점을 수정하려고 만든게 `stringbuffer`, `stringbuilder`이다.
그전에 string 객체가 존재하는데 스트링 리터럴을 사용해야하는 이유가 무엇일까?

> **Q :  Why java uses the concept of string literal?**
>

> A : `String literal concept is used to make Java more memory efficient. As multiple references can point to one value in string pool and no new object will be created if it already exist in string constant pool.`
>

첫번째로 스트링 리터럴이 생긴 이유는 **메모리의 효율성**을 높이기 위해서이다. 리터럴을 사용하면 같은 데이터 값(= contents)은 string pool 내에서 있는 같은 주소를 바라보기 때문에 메모리의 낭비를 막을 수 있다.

만약 new로  스트링 객체를 만든다면 같은 값이라도 다른 주소를 갖는 스트링이 만들어지기때문에 **리터럴을 사용하는것이 훨씬 메모리 효율성이 좋다.**

# 2. == 과 equals

먼저 ==는 기본 자료형에는 값을비교, 참조자료형에서는 객체의 주소값을 비교한다. 따라서 참조자료형의 값을 비교하려면 equals()를  오버라이딩을 해서 사용해야한다.

## 2.1. String에서의 ==과 equals() 의 차이

```java
String  str1= "hello"; //리터럴
String str3 = new String("hello"); //객체 

System.out.println(str1 == str3);  //false 
System.out.println(str1.equals(str3)); //true

```

### == 로 비교했을 때

먼저  `==`로 비교한 결과를 보면 false이다. 이는 `==`는 참조자료형에서 비교할때 객체의 주소값을 비교하기 때문에 당연히 리터럴과 `new` 키워드로 생성한 두 개의 객체는 `Heap 영역`에서 다르게 존재하기 때문에 false가 나온다.

### equals() 로 비교했을 때

`equals()`의 결과를 보면 true로 같다고 나온다. 왜일까?

```java
//Object의 equals()
public boolean equals(Object obj) {
      return (this == obj); //
  }
```

먼저 `Object`의 `equals()`메소드의 코드를 확인하면 `==` 를 사용해 객체의 주소값을 비교한다. 왜냐면 `==`는 참조자료형일때 주소값을 비교하기때문이다.  이러한 `Object`의 `equals()` 오버라이딩한 `String`클래스의 `equals()`를 확인하면 String의 값 하나하나를 비교해서 객체의 주소값이 아닌 값을 비교하도록 재정의했다.

```java
//String 클래스 
public boolean equals(Object anObject) {
        if (this == anObject) { //객체가 같으면 리턴
            return true;
        }
        return (anObject instanceof String aString)
                && (!COMPACT_STRINGS || this.coder == aString.coder)
                && StringLatin1.equals(value, aString.value); //밑의 메소드로 연결
    }

//StringLatin1 클래스
@IntrinsicCandidate
    public static boolean equals(byte[] value, byte[] other) {
        if (value.length == other.length) {
            for (int i = 0; i < value.length; i++) {
                if (value[i] != other[i]) {
                    return false;
                }
            }
            return true;
        }
        return false;
    }
```

따라서 스트링문자열을 비교할때 값을 비교하고 싶다면 `equals()` 를 사용해서 비교해야한다.

만약에 리터럴의 두 객체를 비교하면 어떻게 될까?
```java
String  str1= "hello";
String  str2= "hello";

System.out.println(str1== str2);
System.out.println(str1.equals(str2));
```
당연히 리터럴을 생성하면 값이 같을 때 Heap 영역의 String Constant pool의 같은 값을 참조하므로 true, equals()로 비교해도 값이 같으므로 true가 나온다.