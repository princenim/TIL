# 📌 자바의 hashcode는 무엇이고, 어디에 사용할까?

# 1. hashCode

Object의 `hashCode()` 메소드는 객체의 메모리 주소를 16진수로 hashcode를 리턴한다. 즉 **hashcode는 객체의 주소값을 이용해 만든 객체의 고유한 정수값을 말한다.**

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

먼저 object 클래스의 hashcode의 리턴값을 확인하면 `int`이다. 이 말은 숫자의 범위가 정해져있다는 의미로 int의 범위인 약 21억개의 hashcode가 생길수 있다는 말이다.
하지만 객체는 무한대로 생성이 가능하다. 따라서 object의 hashcode를 사용하면 다른 객체일지라도 같은 hashcode의 값이 생성될 가능성이 존재한다. **즉 hashcode가 무조건 고유한 값이라는 말은 틀리다.**

**그렇다면 이 hashcode는 완벽히 고유하지 않는데 어디에서 사용할까?**

결론부터 말하면 **객체의 동등성(객체가 가지는 값을 확인)을 확인할 때 사용되어 객체를 비교할때 드는 비용을 낮추기 위함이다.**

객체를 비교할 때 객체안의 필드값들이 같아야 완전히 같은 객체라고 정의했을 때 (= 동등성) 클래스 안에 필드값이 100억개가 있다면 모든것을 `equals()` 메소드로 비교하려고 하면 시간이 많이 걸리고 비효율적이다. 따라서 이때 사용하는 것이 `hashcode()`이다.

`hashcode()`를 통해 해시코드값을 만들면 만들 수 있는 해시코드값이 한정되어있기때문에 다른 객체라도 같은 해시코드 값이 나올수 있다. **이말은 즉 해시코드 값이 같을 때 객체가 다를 가능성은 있지만, 해시코드값이 다르다는 것은 무조건 객체의 주소값이  다르다는 것을 뜻한다.**  따라서 `equals()` 메소드를 사용하기 전에 해시코드를 사용해 한번 필터링을 한다면  `equals()` 메소드에서 검사하기전에 빠르게 한번 걸러낼 수 있으며 이를 통해 단순히 `Integer`만 비교하면 되므로 더 효율적으로 동등성을 비교할 수 있다.

**따라서 객체의 동등성을 확인할 때 equals() 와 hashcode()를 동시에 오버라이딩해서 사용해야한다.**

또한 이 hashcode는 checksum과 비슷하다.

### checksum

체크썸의 대표적인 예는 주민등록번호의 마지막 숫자이다. 마지막 숫자는 나머지 숫자의 공식에 의해 만들어진 숫자로 웹사이트에서 주민등록번호 검사를 할 때 서버를 통해 확인하지 않아도 먼저 검증을 해서 확인한 후에 서버에 요청해서 확인을 하는 방식이다. 따라서 서버의 부하를 막을 수 있는 효과가 있으며 인터넷 카드 결제 시 마지막 3자리 입력하는 것도 checksum을 말한다.

이와같이 hashcode와 checksum은 단순히 integer 하나만 비교하면 되므로 코드 상의 성능을 높일 수 있는 좋은 방법이다.

# 2. Hashtable의 hashcode

`Hashcode`는  `Hashtable`의 인덱스를 구할 때 가장 많이 사용된다.
`Person` 이라는 클래스가 존재할때 여기서 필드값이 다 같아야 같은 사람이라고 가정해보자.(동등성)

```java
class Person { //사람이라는 클래스가 존재. 그리고 필드값이 다 같아야 같은 사람이라고 가정할 때
    String name;
    int age;
    int phone;

    public Person(String name, int age, int phone) { //생성자
        this.name = name;
        this.age = age;
        this.phone = phone;
    }
}
```

```java
public class HashCodeTest {
    public static void main(String[] args) {
        Person a = new Person("a", 10, 1234); //다른 메모리 주소를 가진 객체 -> 즉 hashcode도 다름 ->
        Person b = new Person("a", 10, 1234);

        a.hashCode(); // 해시코드 15
        b.hashCode(); // 해시코드 19 -> 객체의 주소가 다르니까

        //해쉬 테이블을 생성 ->
        Map<Person, Integer> map = new HashMap<>(); // 10 elements array , 내부적으로 배열을 생성함.

        //해쉬코드를 인덱스로 사용함.
        // [0, 0, 0, 0, 0, 10, 0, 0, 0, 12] <- [a] where? = hashCode = 15.  15 % 10 = 5,   19 % 10 = 9

        map.put(a, 10);//a의 해쉬코드가 15이므로 15 % 10 = 5 -> 인덱스가 5가 됨 이때 절대 0~9이상의 숫자가 나올 수 없음
        map.put(b, 12); // b의 해쉬코드가 19이므로 19 % 10 = 9 -> 인덱스가 9가 됨
        //따라서 인덱스가 같다면 바로 덮어쓰어지고 그렇지 않다면 다른 인덱스 자리에 값이 추가된다.

        map.get(b); // return 12,   <- 19, 19 % 10 = 9, b의 해시코드 19로 인덱스 9인 배열의 값이 나옴.
        map.get(a); // return 10,  a의 해시코드인 15로 인덱스가 5의 배열의 값이 12가 10이 나옴.
        //따라서 두 객체는 같아야하는데! 이 해시코드를 미리 오버라이딩해서 두 해시코드가 같은 필드가 존재할때 같은 해시코드를 가질수 있게한다.
    }
}

```

그리고 `Person`이라는 클래스를 이용해 같은 값을 넣고 객체를 만들고 해시코드를 확인해보면 다른 해시코드 값이 나온다. 당연히 두 객체는 다른 객체로 주소값이 다르기 때문이고 해당 주소값을 사용해 해시코드를 만들어지기 때문이다.
이제 `HashTable`을 하나 생성해보자. `HashTable`은 `Key-value` 형태로 데이터를 저장하는 자료구조로 빠르게 데이터를 검색할 수 있는 있다. 그리고 이 `Hashtable`은 내부적으로 배열로 구현되어있다.
만약 이 배열이 디폴트 값인 10으로 생성된다고 가정하고 생각해보면 처음에는 0으로 초기화되어있다.

`[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]`

그리고 Hashtable의 값을 저장해야한다면 어디에 저장해야할까?

이때 배열의 인덱스로 사용하는게 `Hashcode`이다. 객체 `a`의 `hashcode()`의 해시코드는 15를 이용해 인덱스로 사용한다. a의 해시코드가 15이므로 `15 % 10 = 5` 즉 이렇게 나머지값을 구하면 0 ~9사이의 값이 나오고 10이상의 값이 나올수 없으니 나머지를 구해 ***인덱스***로 사용하는 것이다.
따라서 인덱스가 5가되고 , b의 경우는 `19 % 10 = 9`가 되어 인덱스 9에 값 12가 저장된다.

`[0, 0, 0, 0, 0, 10, 0, 0, 0, 12]`  마찬가지로 `Hashtable`의 값을 구할 때도 똑같은 방법으로 구한다.

하지만 여기서 중요한점은 우리는 두 객체를 같은 객체라고 생각하고 있어서 `Hashtable`에서 덮어씌어져하는데 다른값으로 취급되어 다른 값으로 저장되었다. 이때  `hashcode()` 오버라이딩을 해야한다.

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

이렇게 `hashcode`()를 오버라이딩해서 재정의해주면 해시코드자체가 같아지므로 `hashtable`을 사용할때 같은 객체로 취급되어 동등성을 비교할 수 있다.

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

분명 위에서  **hashcode는 객체의 주소값을 이용해 만든 객체의 고유한 정수값**으로 말했기때문에 ****리터럴은 heap의 string pool에서 object로 만든 String은 heap영역 만들어져서 다른 해쉬코드를 가질것이라고 예상했다.

하지만 모두 같은 값이 나왔다. 왜일까?

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
        // The hash or hashIsZero fields are subject to a benign data race,
        // making it crucial to ensure that any observable result of the
        // calculation in this method stays correct under any possible read of
        // these fields. Necessary restrictions to allow this to be correct
        // without explicit memory fences or similar concurrency primitives is
        // that we can ever only write to one of these two fields for a given
        // String instance, and that the computation is idempotent and derived
        // from immutable state
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

### 내부 구현 코드

1. 멤버 변수 hash가 있으면 즉, hash code를 이미 계산한적이 있다면 int의 기본값인 0 또는 멤버변수 hash를 반환한다.
    1. `value[]` 가 `final`로 선언되어 `immutable` 즉 불변이다. 따라서 한번 대입하면 변하지 않는다.
2. 계산한 적이 있으면 캐릭터(char)을 앞에서부터 한자씩읽으면서 아스키 코드로 읽어 31을 곱한다 (char은 문자형으로 아스키코드로 표현가능하다.)

### 31을 곱하는 이유?

이펙티브 자바에서는 이렇게 설명한다.

> **31은 소수(1과 자신만을 약수로 가지는 수)이면서 홀수이기 때문에 선택된 값이다.** 만일 그 값이 짝수였고 곱셈 결과가 오버플로되었다면 정보는 사라졌을 것이다. 2로 곱하는 것은 비트를 왼쪽으로 shift하는 것과 같기 때문이다. 소수를 사용하는 이점은 그다지 분명하지 않지만 전통적으로 널리 사용된다. 31의 좋은 점은 곱셈을 **시프트와 뺄셈**의 조합으로 바꾸면 더 좋은 성능을 낼 수 있다는 것이다 (31 * i는 (i << 5) - i 와 같다). 최신 VM은 이런 최적화를 자동으로 실행한다.
>
1. **홀수를 사용하는 이유**

짝수를 곱하면 비트의 오른쪽 값이 항상 0으로 채워진다.하지만 홀수의 경우는 항상 0으로 채워지지않고 1이 생기거나 안 생기면서 비트가 채워진다. 따라서 비트는 한정되어있는데 계속 0으로 채워지면 오버플로우가 발생하므로 홀수를 사용한다.

1. **비트 연산이 훨 빠르다.**

연산에서 곱셈을 하지 않고 시프트 연산과 뺄셈을 함으로써 성능을 더 향상시켰다. `31 * i == (i << 5) - i`
로 같기 때문에 `i`를 왼쪽으로 5번 쉬프트 즉 `2^5` 그리고 `i`를 한번 뺀다.

### 쉬프트 연산이란?

비트를 `<<` 또는 `>>` 으로 이동시키는 것
예를 들어 `8 <<3` 라면 10진수 8의 2진수를 왼쪽으로 3자리 이동한다. 즉, 3자리 이동했다는 것은 3개 비트를 2로 이동했다는 의미니까  `8 * 2^3 = 64`이다. 따라서 `x<<n ⇒ x * 2^n`을 말하고, `x >> n ⇒ x / 2^n`을 말한다.