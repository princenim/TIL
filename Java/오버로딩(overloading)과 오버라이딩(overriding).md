# 📌 오버로딩(overloading)과 오버라이딩(overriding)

`오버로딩`과 `오버라이딩`은 둘다 다른 개념이지만 이름이 비슷해서 혼동이 올 수있다. 그래서 좀더 정확히 정리해보려고한다.

# 1. 오버로딩(overloading)

```java
public class ReferenceOverloading {
    public static void main(String[] args) {
        ReferenceOverloading referenceOverloading = new ReferenceOverloading();

    }

    public void print(int data){

    }

    public void print(String data){

    }

    public void print(int intData, String stringData){

    }

    public void print(String StringData, int intData){
        
    }
}
```

위에 있는 메소드들은 모두 이름이 동일하다 하지만 각 메소드의 매개변수의 종류와 개수, 순서가 다르다. 이렇게 메소드 매개변수의 개수가 같아도 타입의 순서가 다르면 다른 메소드처럼 인식된다. 이렇게 **메소드의 이름을 같도록하고 매개변수만들 다르게 하하는 것을 오버로딩(overloading)이라고 한다.** 가장 대표적인 예시가
`System.out.println()` 이다. 메소드의 이름은 다르지만 매개변수를 다르게 사용해 다양한 타입을 출력할 수있다. 오버로딩은 코드의 가독성을 향상시키고 유연성을 제공한다.

# 2. 오버라이딩(overriding)

**자식클래스에서 부모 클래스에 있는 메소드와 동일하게 선언하는 것을 오버라이딩(overriding)이라고 한다. 접근제어자, 리턴타입, 메소드 이름, 매개 변수 타입 및 개수가 모두 동일해야만 메소드 오버라이딩이라고 한다.**

```java
public class ParentOverriding {
    public ParentOverriding() { //생성자
        System.out.println("Parent overriding constructor");
    }

    public void printName() {
        System.out.println("printName() - parentOverring");
    }
}
```

```java
public class ChildOverriding extends ParentOverriding {

    public ChildOverriding(){ //생성자
        System.out.println("childoverrind constructor");
    }

    @Override
    public void printName(){
        System.out.println("ChildOverriding printName()");
    }
}
```

```java
public class InheritanceOverriding {
    public static void main(String[] args) {
        ChildOverriding child = new ChildOverriding();
        child.printName();
    }
}
```

이렇게 부모 클래스와 자식클래스를 만들고 실행했을때 결과는 다음과 같이 출력된다.

```java
Parent overriding constructor //부모 클래스 생성자
childoverrind constructor //자식 클래스 생성자
ChildOverriding printName()
```

자식 클래스의 객체를 만들고 실행했을때 부모 클래스에 선언되어있는 메소드를 자식클래스에 동일하게 선언하면 자식클래스의 메소드만 실행되는 것을 알 수있다. 동일하게 선언했다라는 말은 ‘동일한 시그니처를 가진다.’라고 표현할 수있다. 시그니처는 메소드 이름과 매개변수의 타입 및 개수를 의미한다.

오버로딩의 경우 자식 클래스 메소드가 부모클래스의 접근 제어자보다 더 확대되는것에는 문제가없지만, 더 작아진다면 컴파일 에러가 발생한다.  `public → protected → package-private → private` 로 오른쪽으로 갈수록 접근 권한이 좁아지므로 주의해야한다.

# 3. 오버로딩과 오버라이딩의 차이점

- `오버로딩` :  확장.  동일한 이름의 메소드를 매개변수를 다르게 여러개 가질 수있게 하기때문에
- `오버라이딩` :  덮어씀.  부모 클래스의 메소드 시그니처를 복제해 자식 클래스에서 새로운 것을 만들어 내어 부모 클래스의 기능은 무시하고 자식 클래스에서 덮어쓰기때문에