# 1. call by value, call by reference

메소드(함수)의 호출 방법은 대표적으로 `call by value`와 `call by reference`가 있다.

## 1.1 call by value

**메소드를 호출할 때 값을 복사해 넘겨주는 방식으로 pass by value라고도 부른다.**

메소드를 호출하는 호출자 (Caller)의 변수와 호출 당하는 수신자(Callee)의 매개 변수는 **복사된 서로 다른 변수**이다.  값만을 전달하기 때문에 수신자의 매개 변수를 수정해도 호출자의 변수에는 아무런 영향이 없다.

## 1.2 call by reference

**참조(주소)를 직접 전달하여 pass by reference 라고도 부른다.**

주소를 직접 넘기기 때문에 호출자의 변수와 수신자의 매개변수는 완전히 동일한 변수이다. 메소드 내에서 파라미터를 수정하면 그대로 원본 변수에도 반영된다.

## 1.3 그래서 자바는?

결론부터 말하자면 자바는 무조건 `call by value`이다.

기본자료형(int, short, long ,float, char, boolean) 아닌 참조자료형 (Array, Class)이 주소값을 넘기는 `call by reference`처럼 보이는데 사실은 주소값이 아닌 참조를 넘기기 때문에 `call by value`이다.

자바에서는 메소드를 호출할 때 매개변수로 지정한 값을  매개변수에 **복사**해서 전달한다. **타입이 기본형일때는 기본형 값을 복사, 참조형일때는 인스턴스의 주소가 복사해 전달한다.**

# 2. 자바의 원시 타입 전달

```java
public class PrimitiveTypeTest {
    public static void main(String[] args) {
        int a = 1; //기본 자료형이니 stack에 저장

        System.out.println(a); //1

        change(a); //
        System.out.println(a); //1
    }

    public static void change(int a) { //이 매개변수와 넘겨받은 변수는 다른 변수임
        a = 3;//a의 값을 변경

    }
}
```

메소드에서 a 값을 변경했는데도 다시 출력했을 때 값이 변하지 않았다 . 이러한 이유는 기본 자료형을 메소드에 전달할때 a라는 값을 change의 메개변수 a에 **복사**해서 넘기기 때문이다. 즉, main에서 생성한 변수 a와 change메소드의 매개변수 a는 다른 값이다.

<img width="951" alt="스크린샷 2024-01-11 오후 3 48 40" src="https://github.com/princenim/TIL/assets/59499600/fd352480-6737-454e-b6fc-fe0e17944783">


1) int형 변수 `a`를 생성
2) `change` 메소드를 호출. 이 때 `stack` 영역에 스택프레임이 생기고 메소드의 매개변수 `a`가 생성 그리고 `a`의 값이 복사되어 매개변수 `a`에 전달된다.
3)  `change` 메소드의 변수 `a`의 값을 3으로 변경. 그리고 이 `change` 메소드가 종료되면 해당 스택프레임은 메모리에서 사라진다.

**즉, 원시타입의 전달은 값만 전달하는 Call by Value이다.**

# 3. 자바의 참조 타입 전달

```java
class User{
    public int age;

    public User(int age){
        this.age = age;
    }
}

public class ReferenceTypeTest {

    void main(){
        User a = new User(10); //변수
        User b = new User(10);

        change(a,b); //호출

        System.out.println(a); //값이 변경됐음

    }

    public void change(User a, User b){ //참조변수 a와 b의 값(주소)가 복사.
        a.age++;

        b= new User(30);
        b.age++;
    }

}
```

![알고리즘-11 2](https://github.com/princenim/TIL/assets/59499600/c3420f70-cb24-4612-9b24-6509d18a50de)

1) 객체 `a`와 `b`를 생성
2) `change` 메소드가 호출되어 `stack` 영역에 스택프레임이 생기고, 매개변수가 참조형이니 **참조변수의 값**(주소)을 매개변수 `a`와 `b`로 복사해서 전달
3) `change` 메소드에서 `a`의 `age`값을 1늘리고, 새로운 객체를 생성 그리고 이 새로운 객체의 주소값을 `change`의 변수 `b`가 참조하도록 한다.
이렇게 `change` 메소드의 호출이 끝나면 스택프레임이 사라지고 결국 main의 변수 `a`,`b`의 값은 11과 20으로 나온다 .

또 다른 참조 타입 전달의 예이다.

```java
class Person{
    int age;
}

public class ReferenceTypeTest2 {
    public static void main(String[] args) {
        Person p = new Person();
        p.age= 10;

        System.out.println(p.age); //10

        test(p); //값 변경

        System.out.println(p.age);
        System.out.println(p);

    }

    public static void test(Person p){ //참조변수의 전달- 참조변수 d의 값(주소)가 매개변수 d에 복사, 주소가 아니라 참조의 값이라고 생각하기

        p.age = 15;
				p =  null;
    }
}
```

![알고리즘-12](https://github.com/princenim/TIL/assets/59499600/9f0481d4-b28c-41cb-a5dc-3efb86c9d0c4)

1) 객체 `p` 생성

2) `change` 메소드가 호출되어 `stack` 영역에 스택프레임이 생기고, 매개변수가 참조형이니 **참조변수의 값**(주소)을 매개변수 `p`이 복사되어 전달된다.

3) `change` 메소드에서 `a`의 `age` 값을 15로 변경하고, `change`메소드의 변수 `p`에 null 값을 넣는다. 따라서

이렇게 `change` 메소드의 호출이 끝나면 스택프레임이 사라지고 `p`의 `age` 값은 15로 출력된다.

즉, 다시말해 자바에서 **참조자료형에서 주소값를 전달하는 것처럼 보여 `call by reference` 처럼 보이지만 주소값!을 넘기는 게 아니라 실제로는 주소값을 가지고 있는 참조 자체를 넘기기 때문에  call by value라고 볼 수 있다.**