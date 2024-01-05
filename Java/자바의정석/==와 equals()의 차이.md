
# 1. == 와 equals()의 차이

## 1.1 ==

기본자료형에서 두 개 의 값이 다른지 비교하는 연산자는 == ,≠ 이다. ==는 같은지를 비교, ≠는 서로 다른지를 비교한다. 이 결과는 무조건 boolean타입이다. 하지만 == 연산자는 참조자료형에서 사용할수없다. 정확히 말하면 == 는 참조자료형의 ‘값’을 비교하지 않고, 객체의 주소값을 비교한다.

- 기본자료형 : 값을 비교
- 참조자료형 : 객체의 주소를 비교

```java
public class MemberDTO {
    public String name;
    public String phone;
    public String email;

    MemberDTO(String name) { //생성자
        this.name = name;
    }
}
```

이런 memberDTO가 있을 때

```java
public class Equals {
    public static void main(String[] args) {
        Equals equals = new Equals();
        //equals.equalMethod();
        equals.equalMethod2();
        //equals.myTest();

    }

    public void equalMethod() {
        MemberDTO obj1 = new MemberDTO("홍"); //참조자료형
        MemberDTO obj2 = new MemberDTO("홍");

        if (obj1 == obj2) { //결과는 두 객체의 주소값이 다르기 때문에 different
            System.out.println("obj1 and obj2 is same");
        } else {
            System.out.println("obj1 and obj2 is different");
        }

    }
}
```

두 참조자료형을 == 로 비교하면 값이 같은데도 불구하고  obj1 and obj2 is different 라는 결과가 나온다. 당연히 == 는 주소값을 비교하기때문이다.

그렇다면 참조자료형에서 값을 비교하려면 어떻게 해야할까?

## 1.2 eqauls()

```java
public void equalMethod2() {
        MemberDTO obj1 = new MemberDTO("홍"); //참조자료형
        MemberDTO obj2 = new MemberDTO("홍");
        if (obj1.equals(obj2)) { //비교 
            System.out.println("obj1 and obj2 is same");
        } else {
            System.out.println("obj1 and obj2 is different");
            //결과는 다르다 왜냐하면 MemberDto 클래스에서 오버라이딩 하지 않았기 때문에
        }
}
```

이렇게 최상위 클래스의 메소드인 equals()로 사용했을떄는 ‘obj1 and obj2 is different’ 라는 결과가 나온다. 그렇다면 참조자료형의 값을 비교하려면 어떻게 해야할까? MemberDTO에서 **equals()메소드를 오버라이딩해야한다.**

```java

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        // 주소값이 같으므로 당연히 true. o는 obj2
        //this는 인스턴스 자신을 가리키는 참조변수
        // -> 즉 클래스가 메모리에 올라간 상태를 가리키는 참조변수
        // -> 그럼다면 MemeberDto의 객체인 obj1과 obj2

        if (o == null || getClass() != o.getClass()) return false;
        //객체가 null이거나 클래스의 종류가 다르면 false

        MemberDTO memberDTO = (MemberDTO) o; //같은 클래스이므로 형변환 실행

        //각 인스턴스 변수가 같은지 비교하는 작업 수행
        //인스턴스 변수 name과 객체로 넘어온 obj2의 name을 비교
        //equals()는 참조자료형일떄 값을 비교, 기본자료형도 값을 비교한다.
        return Objects.equals(name, memberDTO.name)
                && Objects.equals(phone, memberDTO.phone)
                && Objects.equals(email, memberDTO.email);

    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, phone, email);
    }
```

이렇게 MemberDTO에서 `equals()`를 오버라이딩을 했을 때 다시 위의 equalMethod2()를 실행하면, ‘obj1 and obj2 is same’ 이라는 값이 나온다!

그렇다면 왜 equals()를 오버라이딩을 하지않고,` equals()`를 사용했을 때는 ’obj1 and obj2 is different’라는 결과가 나온것일까?

```java
//Object의 equals()
public boolean equals(Object obj) {
        return (this == obj);
    }
```

그 이유는 **최상위 클래스 Object의 `equals()`는 코드를 확인했을때 내부적으로 비교연산자인 ==와 동일한 결과를 리턴하기 때문이다.** 따라서 **참조자료형의 주소값이 아닌 값을 비교하고자하면 사용하는 클래스에 equals()메소드를 오버라이딩을 해서 두 인스턴스가 동등하다고 따로 정의를 해서 equals() 메소드를 사용해야한다.**

단! `equals()메소드`를 오버라이딩할떄 hashCode()메소드도 같이 오버라이딩을 해야한다. 왜일까?

## 1.3 equals()와 hashCode()를 같이 오버라이딩 해야 하는 이유

먼저 Object의 `hashcode()` 메서드는 기본적으로 객체의 메모리 주소를 16진수로 리턴해준다.  객체에 메모리 주소를 이용해 해시코드를 만들어주기때문에 객체마다 다 다른 값을 가지고 있다. 정확히 말하면 주소값은 아니고 주소값으로 만든 고유한 숫자값이다.

**결론부터 말하면 equals 메소드와 hashcode메소드를 같이 오버라이딩해야하는 이유는 hash값을 사용하는 오버라이딩을 같이 하지않으면 Colletion(HashSet, HashMap, HashTable) 을 사용할 때 문제가 발생한다.**

참고로 hash 자료구조는 key값을 통해 value를 꺼내거나, 동일한 객체는 중복해서 추가할수없는 등의 경우에 따라서 서로 객체가 같은지 다른지 비교한다.

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

중복 허용을하는 컬렉션 ArrayList의 길이는 2가 나온다. 그렇다면 중복허용을 하지 않는 HashSet의 클래스는 길이는 `equals()`로 확인했을 때 객체가 동일하니 1이 나올지 알았지만 2가 나왔다.

왜 그럴까?
다음은 **중복을 허용하지 않는** hash 자료구조의 객체 중복 여부 판단 과정이다. 

![hash](https://github.com/princenim/TIL/assets/59499600/1a28e830-9876-452b-8c1e-272ca603b976)

1. `hashCode()`비교를 통해 두 객체의 주소값을 비교한다.

   1-1. 두 객체의 리턴값이 같으면 다르면 당연히 다른 객체
   
   1-2. 두 객체의 리턴값이 일치하면 equals()를 통해 한번더 확인. `equals()`는 두 객체의 주소값이 아닌 값을 확인하므로 이때 이 값이 다르면 당연히 다른객체, 값이 같으면 동일 객체로 판단한다.


`hashCode()` 메소드의 리턴값이 일치하고, `equals` 메소드의 리턴값도 일치했을때 완전히 같은 객체라고 판단한다. 하지만 위의 코드에서는 `hashCode` 메소드가 오버라이딩되어있지 않기 때문에 Object 클래스의 hashCode를 사용되어 객체의 고유한 주소 값을 int로 반환했다. 
따라서 두 Car객체는 오버라이딩된 `equals()` 메서드의 비교도 전에 서로 다른 hashCode의 리턴값을 통해서 다른 객체로 판단한 것이다.
따라서 이런 잘못된 결과를 방지하기 위해서 `euqals()`를 오버라이딩할떄 hashCode 재정의하는 것이 좋다.

## 1.4 동일성과 동등성

### 동일성

**동일성은 두 객체가 완전히 같은 경우를 완전히 같은 것을 의미한다 즉 같은 주소값을 갖는다.** == 연산자로 판별할 수있다.
참고로 기본자료형은 객체가 아니라 주소가 없으므로 `==` 연산자를 사용했을 때 내용이 같으면 동일하다고 한다.

### 동등성
**동등성은 두 객체의 주소값이 다르지만, 가지고 있는 값이 같음을 의미한다.**` equals` 으로 비교한다.