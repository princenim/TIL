# **📌** 자바의 hashCode

# 1. hashCode

 Object의 `hashCode()` 메소드는 객체의 메모리 주소를 16진수로 hashcode를 리턴한다. 즉 **hashcode는 객체의 주소값을 이용해 만든 객체의 고유한 정수값을 말한다.** 

```java
public static void main(String[] args) {
        HI hi = new HI();
        System.out.println(hi.hashCode()); //932583850
}
```

만약 어떤 두 객체가 서로 동일하다면 `hashCode()` 값은 무조건 동일해야만한다. 따라서 `equals()`로 메소드를 오버라이딩을 할때 `hashCode()` 도 오버라리딩해서 동일한 결과를 나오도록 만들어야만 한다. 

### Object의 equals()와 hashCode()를 같이 오버라이딩 해야 하는 이유

**결론부터 말하면 equals 메소드와 hashcode메소드를 같이 오버라이딩해야하는 이유는 hash값을 사용하는 오버라이딩을 같이 하지않으면 Colletion(HashSet, HashMap, HashTable) 을 사용할 때 문제가 발생하기 때문이다.**  참고로 hash 자료구조는 key값을 통해 value를 꺼내거나, 동일한 객체는 중복해서 추가할수없는 등의 경우에 따라서 서로 객체가 같은지 다른지 비교한다.

```java
public class Car {
    String name;

    Car(String name){//생성자
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Car car = (Car) o;
        return Objects.equals(name, car.name);
    }

}
```

Car클래스가 있고 equals()만 오버라이딩하고 hashcode는 오버라이딩하지 않았을때

```java
public class CarTest {
    public static void main(String[] args) {
        Car car1 = new Car("HI");
        Car car2 = new Car("HI");
        System.out.println(car1.equals(car2)); //equals를 오버라이딩해서 true

        //중복허용하는 컬렉션
        ArrayList<Car> cars = new ArrayList<>();
        cars.add(car1);
        cars.add(car2);
        //System.out.println(cars.size());///2

        //종복을 허용하지 않는 컬렉션
        HashSet<Car> cars2 = new HashSet<>();
        cars2.add(car1);
        cars2.add(car2);
        System.out.println(cars2.size());//2
        //hashcode 를 오버라이딩하지않았을떄는 2이 나옴.

    }
}
```

중복 허용을하는 컬렉션 `ArrayList`의 길이는 2가 나온다. 그렇다면 중복허용을 하지 않는 HashSet의 클래스는 길이는 `equals()`로 확인했을 때 객체가 동일하니 1이 나올지 알았지만 2가 나왔다.

왜 그럴까? 다음은 **중복을 허용하지 않는** hash 자료구조의 객체 중복 여부 판단 과정이다.

<img width="725" alt="스크린샷 2024-01-15 오후 11 48 04" src="https://github.com/princenim/TIL/assets/59499600/f0e4457b-c3c4-4a66-8354-f7160b4d8254">


1. `hashCode()`비교를 통해 두 객체의 주소값을 비교한다.
    
    1-1. 두 객체의 리턴값이 같으면 다르면 당연히 다른 객체
    
    1-2. 두 객체의 리턴값이 일치하면 equals()를 통해 한번더 확인. `equals()`는 두 객체의 주소값이 아닌 값을 확인하므로 이때 이 값이 다르면 당연히 다른객체, 값이 같으면 동일 객체로 판단한다.
    

`hashCode()` 메소드의 리턴값이 일치하고, `equals` 메소드의 리턴값도 일치했을때 완전히 같은 객체라고 판단한다. 하지만 위의 코드에서는 `hashCode` 메소드가 오버라이딩되어있지 않기 때문에 Object 클래스의 hashCode를 사용되어 객체의 고유한 주소 값을 int로 반환했다. 따라서 두 Car객체는 오버라이딩된 `equals()` 메서드의 비교도 전에 서로 다른 hashCode의 리턴값을 통해서 다른 객체로 판단한 것이다. 따라서 이런 잘못된 결과를 방지하기 위해서 `euqals()`를 오버라이딩할때 hashCode 재정의하는 것이 좋다.

# 2. String의 hashCode

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

코드를 보면 String 내부의 char 원소를 이용해 새로운 int값을 만들어 반환한다.
이렇게 재정의한 이유는 위에서 알수있듯이 해쉬코드를 중복을 허용하지 않은 HashMap이나 HashSet에서 사용하기 위해서이다.  같은 문자열을 갖는 두 객체의 hashcode가 다르다면 hashcode를 먼저 확인하는 의미가 없으니 미리 재정의함으로써 같은 문자열을 갖는 객체에 대해 같은 hashcode를 갖도록 만든것이다. 

**즉, 주소값은 다르나 같은 문자열을 가지면 같은 해쉬코드를 가질 수있다.**

### 내부 구현 코드

1. 멤버 변수 hash가 있으면 즉, hash code를 이미 계산한적이 있다면 int의 기본값인 0 또는 멤버변수 hash를 반환한다. 
    1. value[] 가 final로 선언되어 immutable 즉 불변이다. 따라서 한번 대입하면 변하지 않는다. 
2. 계산한 적이 있으면 문자열을 앞에서부터 한자씩읽으면서 아스키 코드로 변환해 처리한다. 
    1. 계산한 값에 31을 곱하고 리턴한다
    

### 31을 곱하는 이유?

이펙티브 자바에서는 이렇게 설명한다. 

> **31은 소수(1과 자신만을 약수로 가지는 수)이면서 홀수이기 때문에 선택된 값이다.** 만일 그 값이 짝수였고 곱셈 결과가 오버플로되었다면 정보는 사라졌을 것이다. 2로 곱하는 것은 비트를 왼쪽으로 shift하는 것과 같기 때문이다. 소수를 사용하는 이점은 그다지 분명하지 않지만 전통적으로 널리 사용된다. 31의 좋은 점은 곱셈을 **시프트와 뺄셈**의 조합으로 바꾸면 더 좋은 성능을 낼 수 있다는 것이다(31 * i는 (i << 5) - i 와 같다). 최신 VM은 이런 최적화를 자동으로 실행한다.
> 

즉, 곱셈보다 bit shift가 더 빠르기 때문이다. `31 * i == (i << 5) - i`
 (위의 연산의 의미는 i의 비트열을 왼쪽으로 5만큼 이동하라는 뜻이다.)

위와 같이 **31을 곱하는것을 정수를 왼쪽으로 5번 shift해서 빼는 결과와 같다.** 좀더 자세하게 설명하면 31= 32-1이다. 32는 2의5승이면 즉 31 = 2^5 -1 이다. 
따라서 2^5는 왼쪽으로 5번 shift한 결과와 같아. 결론적으로 32 = (<<5)-1 이된다.  2진수 정수의 특성상 왼쪽으로 한칸식 비트열을 움직이면 2의 배수곱이 된다. 

즉, 곱셈보다 비트연산의 성능이 훨씬 좋기때문에 hashCode 31값을 선택한 것이다.