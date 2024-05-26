# 📌 객체 지향 프로그래밍(Object-Oriented Programming, OOP)

# 1. 객체 지향 프로그래밍

객체 지향 프로그래밍이란, 현실 세계의 사물을 객체로 정의하고 이러한 객체들 간의 상호 작용을 모델링하여 프로그래밍하는 것을 말한다. 객체 지향의 4가지 특징은 `캡슐화`, `상속`, `추상화`, `다형성`이다.

## 1.1 캡슐화 (**Encapsulation**)

**캡슐화는 데이터와 메소드를 하나의 단위로 묶어 외부에서 직접 접근하지 못하도록 보호하는 개념이다.** 따라서 객체의 내부 구현을 숨기고, 객체간의 결합도를 낮춰 코드의 유지보수와 재사용성을 높일 수 있다.

```java
public class BankAccount {
    private double balance;

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        } else {
            System.out.println("Deposit amount must be positive.");
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        } else {
            System.out.println("Invalid withdrawal amount.");
        }
    }
}
```

다음과 같은 장점이 존재한다.

1. 내부 구현 숨기기 : balance의 직접 접근을 막아 잘못된 접근으로 발생할 수 있는 오류를 방지할수있다.
2. 데이터 무결성 유지: withdraw와 deposit에서 유효성 검사를 수행하여 데이터의 무결성을 유지할 수 있다.
3. 결합도 낮추기 : 외부 클래스는 BankAccount의 내부 구현을 알 필요가 없으며, 공개된 메소드를 통해서만 상호작용하여 결합도를 낮출 수 있다.

## 1.2 상속(Inheritance)

객체 지향 프로그래밍에서 **상속은 상위 클래스의 특성을 하위 클래스에서 상속(특정 상속)하고 거기에 더해 필요한 특성을 추가, 즉 확장해서 사용하는 것이다.** 즉, 조직도나 계층도의 개념이 아닌 분류도다. 자바에서 상속을 의미하는 단어는 extends(확장)이다. 상속 관계을 표현하는 문장은 `하위 클래스 is a kind of 상위클래스`이다. 펭귄 is a kind of 조류( 펭귄은 조류의 한 분류다) 를 말한다.

그렇다면 자바에서 인터페이스는 어떤 관계를 나타낼까? 답은 `구현클래스 is able to 인터페이스`다. 즉 구현클래스는 인터페이스할 수 있다. 고래는 헤엄칠 수 있다. 라는 의미이다. 그래서 자바 API를 보면 be able to 형식의 인터페이스를 볼 수있다. `Serializable`, `Comparable`, `Runnable`

## 1.3 추상화(Abstraction )

먼저 추상화의 일반적인 뜻은 ‘구체적인 것을 분해해서 관심 영역에 대한 특성만을 가지고 재조합하는 것’을 말한다. IT용어를 이용해 바꿔보면 다음과 같다 . ‘구체적인 것을 분해해 관심 영역(어플리케이션 경계)’에 있는 특성만 가지고 조합하는 것’ 즉 모델링을 말한다.

다시 말하면 객체지향의 추상화는 모델링을 말하며 클래스 설계에서 추상화가 사용된다. 즉, **객체 지향에서 추상화의 결과는 클래스다.** 따라서 추상화를 통해 설계된 클래스는 구체적인 객체를 생성하는 틀이 되며 객체 간의 상호작용을 정의하는데 사용되어 코드의 재사용성을 높인다.

## 1.4 다형성(Polymorphism)

**다형성은 객체 지향 프로그래밍에서 하나의 기능을 여러가지로 구현하거나 표현하는 것을 말한다.** 다형성을 사용해서 코드의 재사용성과 유연성을 높일 수 있다.

자바에서는 대표적으로 `오버로딩`과 `오버라이딩`이 있다. 오버라이딩은 재정의로 상위 클래스의 메소드를 같은 이름, 같은 매개변수로 새롭게 정의하는 것을 말한다. 이를 이용하면 하나의 상위클래스를 상속받는 여러 하위 클래스들이 같은 이름으로 다른 기능을 하는 메소드를 만들 수 있다.  오버로딩은 복수정의로 같은 이름의 메소드를 이름은 같고 , 매개변수를 다르게 해 정의하는 것을 말한다.