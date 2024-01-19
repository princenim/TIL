# **📌** 자바의 hashcode는 무엇이고, 어디에 사용할까?

# 1. hashCode

Object의 `hashCode()` 메소드는 객체의 메모리 주소를 16진수로 `hashcode`를 리턴한다. 즉 **hashcode는 객체의 주소값을 이용해 만든 객체의 고유한 정수값을 말한다.**

```java
		/**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     */
    @IntrinsicCandidate
    public native int hashCode();
```

```java
public static void main(String[] args) {
        HI hi = new HI(); //객체 생성
        System.out.println(hi.hashCode()); //932583850
}
```

먼저 `Object` 클래스의 `hashCode()`의 리턴값을 확인하면 `int`이다. 이 말은 숫자의 범위가 정해져있다는 의미로 `int`의 범위인 약 21억개의 `hashcode`가 생길수 있다는 말이다.

하지만 객체는 무한대로 생성이 가능하다. 따라서 Object의 hashCode를 사용하면 다른 객체일지라도 같은 hashcode의 값이 생성될 가능성이 존재한다. **즉 hashcode가 무조건 고유한 값이라는 말은 틀리다.**

**그렇다면 이 hashcode는 완벽히 고유하지 않는데 어디에서 사용할까?**

## 1.1 해시코드의 존재이유

결론부터 말하면 **객체의 동등성(객체가 가지는 값을 확인)을 확인할 때 사용해 객체를 비교할때 드는 비용을 낮출 수 있다.**

객체를 비교할 때 객체안의 필드값들이 같아야 완전히 같은 객체라고 정의했을 때 (= 동등성) 클래스 안에 필드값이 100억개가 있다면 모든것을 `equals()` 메소드로 비교하면 시간이 많이 걸리고 비효율적이다.  이때 hashCode()를 사용하여 비교 성능을 올릴 수 있다.

앞에서 언급했다싶이 `hashcode()` 를 통해 만들어진 해시코드값이 한정되어있기때문에 다른 객체라도 같은 해시코드 값이 나올수 있다. **이말은 즉 해시코드 값이 같을 때 객체가 다를 가능성은 있지만, 해시코드값이 다르다는 것은 무조건 객체의 주소값이  다르다는 것을 뜻한다.**  따라서 `equals()` 메소드를 사용하기 전에 해시코드를 사용해 한번 필터링을 한다면  `equals()` 메소드에서 검사하기전에 빠르게 한번 걸러낼 수 있으며 이는 단순히 **`Integer`만 비교하면 되므로 훨씬 효율적이다.** 

**따라서 객체의 동등성을 확인할 때 효율적으로 사용하려면 equals() 와 hashcode()를 동시에 오버라이딩하는 것이 좋다.**

이러한 방식은 `checksum` 방식과 비슷하다.

### checksum

체크썸의 대표적인 예는 주민등록번호의 마지막 숫자이다. 마지막 숫자는 나머지 숫자의 공식에 의해 만들어진 숫자로 웹사이트에서 주민등록번호 검사를 할 때 서버를 통해 확인하지 않아도 먼저 검증을 해서 확인한 후에 서버에 요청해서 확인을 하는 방식이다. 따라서 서버의 부하를 막을 수 있는 효과가 있으며 인터넷 카드 결제 시 마지막 3자리 입력하는 것도 `checksum`을 말한다.

이와같이 `hashcode`와 `checksum`은 단순히 `integer` 하나만 비교하면 되므로 코드 상의 성능을 높일 수 있는 좋은 방법이다.

# 2. Hash Collection의 hashcode()와 equals()

`HashCode()`는  `HashSet`,`HashMap`, `HashTable`과 같은 `Hash Collection`에서 주로 사용된다. 그리고 해싱(hashing) 기법에 사용되는 해시함수를 구현한 것이다.

먼저 공통적으로 `Hash Collection` 은 `add,put`과정에서 내부적으로 값이 같은지 비교할때 그림과 같은 과정을 거친다.

<img width="749" alt="스크린샷 2024-01-19 오후 1 53 32" src="https://github.com/princenim/TIL/assets/59499600/e9b92a9e-d5b0-4bda-8baf-29b3ff45835d">


1. 데이터가 추가될때 `hashCode()`비교를 통해 두 객체의 해시코드를 비교한다.

   1-1. 두 객체의 해시코드 값이 다르면 당연히 다른 객체라고 판단한다.

   1-2. 두 객체의 해시코드 값이 같으면  `equals()`를 통해 한번더 확인한다. . `equals()`는 두 객체의 주소값이 아닌 값을 확인하므로 이때 이 값이 다르면 당연히 다른객체, 값이 같으면 동일 객체로 판단한다.

## 2.1 equals()와 hashCode()를 같이 오버라이딩 해야 하는 이유

![Hashtable](https://github.com/princenim/TIL/assets/59499600/ccf19a0c-68be-4d2b-a6c1-75cb27db1aec)
```java
public class Person {
    String name;
    int age;

    Person(String name, int age){
        this.name = name;
        this.age = age;
    }
}
```

`Person` 이라는 클래스가 존재할때 여기서 필드값이 다 같아야 같은 사람이라고 가정해보자.(동등성)

```java
public class HashCodeTest {
    public static void main(String[] args) {
        Person a = new Person("a", 10, 1234); //다른 메모리 주소를 가진 객체
        Person b = new Person("a", 10, 1234);

        a.hashCode(); // 해시코드 15
        b.hashCode(); // 해시코드 19 -> 객체의 주소가 다르니까

        //해쉬 테이블을 생성
        Map<Person, Integer> map = new HashMap<>(); // 10 elements array , 내부적으로 배열을 생성함.

        //해쉬코드를 인덱스로 사용함.
        // [0, 0, 0, 0, 0, 10, 0, 0, 0, 12] <- [a] where? = hashCode = 15.  15 % 10 = 5,   19 % 10 = 9

        map.put(a, 10);//a의 해쉬코드가 15이므로 15 % 10 = 5 -> 인덱스가 5가 됨 이때 절대 0~9이상의 숫자가 나올 수 없음
        map.put(b, 12); // b의 해쉬코드가 19이므로 19 % 10 = 9 -> 인덱스가 9가 됨

        map.get(b); // return 12,   <- 19, 19 % 10 = 9, b의 해시코드 19로 인덱스 9인 배열의 값이 나옴.
        map.get(a); // return 10,  a의 해시코드인 15로 인덱스가 5의 배열의 값이 12가 10이 나옴.

				System.out.println(map.size()); //2
    }
}

```

먼저 `Person`이라는 클래스를 이용해 같은 값을 넣고 객체를 만들고 해시코드를 확인해보면 다른 해시코드 값이 나온다. 당연히 두 객체는 다른 객체로 주소값이 다르기 때문이고 해당 주소값을 사용해 해시코드를 만들어지기 때문이다.

이제 `HashMap`을 하나 생성해보자. `HashMap`은 `Key-value` 형태로 데이터를 저장하는 자료구조로 빠르게 데이터를 검색할 수 있다. 그리고 이 `HashMap`은 내부적으로 `배열`로 구현되어있다.

만약 이 배열이 디폴트 값인 10으로 생성된다고 가정했을때 처음에는 0으로 초기화되어있다. `[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]`

그리고 `HashMap`안에 값을 어떻게 저장될까?

해시코드를 인덱스로 사용해서 값을 저장한다.  해시테이블은 각각의 `key`값에 `해시함수`를 적용해 배열에 고유한 `index`를 생성하고, 이 `index`를 활용해 값을 저장하거나 검색한다. 이때 사용하는 것이 이때 사용하는 해시함수가 `hashcode()`이다.

키 값인 객체 `a`의 `hashcode()`의 리턴값인 해시코드 15를 이용하여  `15 % 10 = 5`  나머지값을 구하면 0~ 9 사이의 값이 나오고 10 이상의 값이 나올수 없으니 나머지를 구해 ***인덱스***로 사용한다.  따라서a의 인덱스는 5가 되어 인덱스 5에 12가 저장, b는 `19 % 10 = 9`가 되어 인덱스 9에 값 12가 저장된다.  배열은 `[0, 0, 0, 0, 0, 10, 0, 0, 0, 12]`   이렇게 저장된다. 마찬가지로 `Hashtable`의 값을 구할 때도 똑같은 방법으로 구한다.

하지만 여기서 중요한점은  `a`와 `b` 두 객체를 같은 객체라고 생각하기때문에  `HashMap`에서 `b`의 `value`인 `12`만 저장되어야 했는데 두개의 값이 저장되어 길이가 2가 나왔다.

이는 위의 내부적으로 `hashcode()` → `equals()` 과정를 통해 객체가 같은지 확인하는데 위의 a와 b의 `hashcode()`의 결과가 15, 19로 다르게 나왔기 때문에 다른 객체로 판별되어 hashMap의 길이가 2가 된것이다.

따라서 이는 두 객체의 해시코드를 같게 하기위해서 `hashcode()` 를 오버라이딩을 해야한다.

```java
class Person { //사람이라는 클래스가 존재. 그리고 필드값이 다 같아야 같은 사람이라고 가정할 때
    String name;
    int age;
    int phone;

    public Person(String name, int age, int phone) {
        this.name = name;
        this.age = age;
        this.phone = phone;
    }

    @Override
    public int hashCode() { //추가
        return name.hashCode() + age + phone;
    }
}
```

이렇게 `hashcode`()를  오버라이딩을 하면 해시코드자체가 같아지기 때문에 같은 인덱스를 가지게 된다.

```java
System.out.println(map.size()); //2
```

하지만 `hashMap`의 길이를 출력하면 `a`와 `b`가 다른 객체로 취급되어 `hashmap`의 사이즈가 2로 나온다.

이는 `equals()` 오버라이딩 하지 않았기 때문이다. `hashcode`는 같지만 해당 `hashcode`의 버킷(Linkedlist) 안에서 `equals()`로 비교할때 동일한 객체를 찾지 못 했기 때문에 다른객체로 판별되어 `HashMap` 안에  추가된것이다.

```java
public class Person {
    String name;
    int age;

    Person(String name, int age){
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) { //추가
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() { //추가
        return name.hashCode() + age ;
    }
}
```

따라서 equals()를 오버라이딩해서 동일한 객체로 판단할 수 있게 해주어야 길이가 1이 나온다.

또한 hashcode는 한정되어있기때문에 위의 예시와 다르게 다른객체임에도 불구하고 같은 해시코드가 나올 수 있다. 따라서 hashcode() 같다고 판별되어도 후에  equals() 메소드를 다른 객체임을 확인할 수 있으므로  **해싱기법을 사용하는 HashMap 이나 HashSet같은 클래스에 저장할 객체라면 반드시 equals()와 hashcode()를 같이 오버라이딩 해줘야한다.**
(여기서 주의할 점은 equals()로 다른 객체임을 확인한다는것은 key 한정이다. value와는 상관없다.)

### 정리

equals()를 오버라이딩할때 무조건 hashcode를 오버라이딩 해야하는 이유

1. hashcode를 사용해서 한번 검사를 할수있으므로 성능을 향상키시는데 도움을 준다.
2.  `Hash collection`에서 사용시 `hashcode()` → `equals()` 비교를 한다. `hashcode`로 검증가능한건 ‘**두 객체가 동일하지 않을때**’뿐이고, hashcode가 같을때는 두 객체가 ‘**같을 수도 다를수도 있기때문에**’ 꼭 `equals()`를 오버라이딩해야한다.
3. 자바 API 문서에서 위와같이 명시한다.
   > 어떤 두개의 객체에 대하여 equals() 메소드를 사용하여 비교한 결과가 true 일경우 두 객체의 hashCode() 메소드를 호출하면 동일한 int값을 리턴해야만한다.


# 3. String은 어떻게 Hashcode를 생성할까?

```java
public static void main(String[] args) {

      String a = "hello world";
      String b = "hello world";
      String c = new String("hello world");

      System.out.println(a.hashCode()); //1794106052
      System.out.println(b.hashCode()); //1794106052
      System.out.println(c.hashCode()); //1794106052
        
  }
```

String 리터럴과 new를 이용한 object로 String을 생성하고 hashcode를 출력하면 다음과 같은 결과가 나온다.

분명 위에서  **hashcode는 객체의 주소값을 이용해 만든 객체의 고유한 정수값**으로 말했기때문에 리터럴은 heap의 string pool에서 object로 만든 String은 heap영역 만들어져서 다른 해쉬코드를 가질것이라고 예상했다.

하지만 모두 같은 값이 나왔다. 왜일까?

## 3.1 내부 구현 코드

```java
//openjdk 17.0.6

@Stable
    private final byte[] value; //문자열 저장
		private int hash; // Default to 0 //해쉬값

/**
     * Returns a hash code for this string. The hash code for a
     * {@code String} object is computed as
     * <blockquote><pre>
     * s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
     * </pre></blockquote>
     * using {@code int} arithmetic, where {@code s[i]} is the
     * <i>i</i>th character of the string, {@code n} is the length of
     * the string, and {@code ^} indicates exponentiation.
     * (The hash value of the empty string is zero.)
     *
     * @return  a hash code value for this object.
     */
    public int hashCode() {
        int h = hash;
        if (h == 0 && !hashIsZero) {
            h = isLatin1() ? StringLatin1.hashCode(value)
                           : StringUTF16.hashCode(value);
            if (h == 0) {
                hashIsZero = true;
            } else {
                hash = h;
            }
        }
        return h;
    }

//StringLatin1.hashCode
public static int hashCode(byte[] value) {
        int h = 0;
        for (byte v : value) {
            h = 31 * h + (v & 0xff);
        }
        return h;
  }
```

결론부터 말하면  String 클래스의 `hashCode()`는 `Object`의 `hashCode()`를 **이미** 재정의했기때문이다.

코드를 보면 String 내부의 `char` 원소를 이용해 새로운 int값을 만들어 반환한다. 즉 객체의 주소와 상관없이 같은 문자열을 가지면 같은 해시코드값을 리턴한다. 이 또한 동등성을 확인할 때 사용할 수 있다.

1. 멤버 변수 hash가 있으면 즉, hash code를 이미 계산한적이 있다면 int의 기본값인 0 또는 멤버변수 hash를 반환한다.
   1. `value[]` 가 `final`로 선언되어 `immutable` 즉 불변이다. 따라서 한번 대입하면 변하지 않는다.
2. 계산한 적이 있으면 캐릭터(char)을 앞에서부터 한자씩읽으면서 아스키 코드로 읽어 31을 곱한다 (char은 문자형으로 아스키코드로 표현가능하다.)

### 31을 곱하는 이유?

이펙티브 자바에서는 이렇게 설명한다.

> **31은 소수(1과 자신만을 약수로 가지는 수)이면서 홀수이기 때문에 선택된 값이다.** 만일 그 값이 짝수였고 곱셈 결과가 오버플로되었다면 정보는 사라졌을 것이다. 2로 곱하는 것은 비트를 왼쪽으로 shift하는 것과 같기 때문이다. 소수를 사용하는 이점은 그다지 분명하지 않지만 전통적으로 널리 사용된다. 31의 좋은 점은 곱셈을 **시프트와 뺄셈**의 조합으로 바꾸면 더 좋은 성능을 낼 수 있다는 것이다 (31 * i는 (i << 5) - i 와 같다). 최신 VM은 이런 최적화를 자동으로 실행한다.
>
1. **홀수를 사용하는 이유** : 짝수를 곱하면 비트의 오른쪽 값이 항상 0으로 채워진다.하지만 홀수의 경우는 항상 0으로 채워지지않고 1이 생기거나 안 생기면서 비트가 채워진다. 따라서 비트는 한정되어있는데 계속 0으로 채워지면 오버플로우가 발생하므로 홀수를 사용한다.

1. **비트 연산이 훨 빠르다.** : 연산에서 곱셈을 하지 않고 시프트 연산과 뺄셈을 함으로써 성능을 더 향상시켰다. `31 * i == (i << 5) - i` 로 같기 때문에 `i`를 왼쪽으로 5번 쉬프트 즉 `2^5` 그리고 `i`를 한번 뺀다.

### 쉬프트 연산이란?

비트를 `<<` 또는 `>>` 으로 이동시키는 것

예를 들어 `8 <<3` 라면 10진수 8의 2진수를 왼쪽으로 3자리 이동한다. 즉, 3자리 이동했다는 것은 3개 비트를 2로 이동했다는 의미니까  `8 * 2^3 = 64`이다. 따라서 `x<<n ⇒ x * 2^n`을 말하고, `x >> n ⇒ x / 2^n`을 말한다.