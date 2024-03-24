# 📌final 키워드

# 1. final

`final` 예약어는 `클래스`, `메소드`, `변수`에 선언할 수있다. 

## 1.1 클래스에 final 선언

```java
public final class FinalClass{

}

```

클래스에 `final`을 선언하면 상속을 해줄 수없다. `String` 클래스같이 더 이상 확장해서는 안 되는 클래스 , 누군가 이 클래스를 상속 받아서 내용을 변경해서는 안 되는 클래스를 선언할 때 `final`로 선언하면 된다. 

## 1.2 메소드에 final로 선언

```java
public abstract class FinalMethodClass {

    public final void printLog(String data){
        System.out.println("data=" +data);
    }
}
```

메소드를 `final`로 선언하면더 이상 `Overriding` 할수 없다. 위와같이 메소드를 `final`로 선언한다면 `FinalMethodClass`를 상속받는 클래스에서 `printLog()` 를 오버라이딩할 수 없다.

## 1.3 변수에 final 선언

변수에 `final`을 사용하는 것은 클래스와, 메소드에 사용하는 것과 약간 다르다. 변수에 `final`을 사용하면 **“더 이상 바꿀 수없다”**는 뜻이다. 따라서 `인스턴스변수`나 `static으로 선언된 클래스 변수`는 선언과 함께 값을 지정해야만한다.

인스턴스 변수와 클래스변수, 그리고 지역변수는 다음과 같다. 

```java
public class test {

    int instanceVariable; //인스턴스변수 -> 클래스 인스턴스를 생성할때 생성
    static int classVariable; //클래스변수 (static변수, 공유변수)

    void method(){
        int localVariable; //지역변수
    }
}
```

```java
public class FinalVariable {
    final int instanceVariable = 1;
}
```

만약에 `인스턴스 변수`에 `final`를 사용한다면 다음과 같이 선언해야한다. 변수 생성과 동시에 초기화를 해야만 컴파일 시 에러가 발생하지 않는다. 

```java
public class FinalVariable {
    final int instanceVariable = 1;

    public void method(final int parameter){
        final  int localVariable;
    }
}
```

하지만 매개 변수나 지역변수를 `final`로 선언하는 경우에는 반드시 초기화를 할 필요가 없다. 왜냐하면 `매개변수`는 이미 초기화가 되어서 넘어왔고, `지역변수`는 메소드를 선언하는 중괄호 내에서만 참조되므로 다른 곳에서 변경할 일이 없다. 따라서 컴파일할때 전혀 문제가 없다. 

```java
public class FinalVariable {
    final int instanceVariable = 1;

    public void method(final int parameter){
        final int localVariable;
        localVariable = 3;
        localVariable= 5; //컴파일 에러 발생
    }
}
```

이렇게 한번 값을 할당한 변수에 다시 값을 할당하려고하면 컴파일 에러가 발생한다. 즉, `final`은 변하지 않는 값에 `final`을 선언하면 매우 유용하다. 

그렇다면 `참조자료형`에도 동일하게 적용될까?

```java
public class FinalReferenceType {

    final MemberDto dto = new MemberDto(); //인스턴스 변수를 final로 선언
    
    public static void main(String[] args) {
        FinalReferenceType referenceType = new FinalReferenceType();
        referenceType.checkDto();
    }

    public void checkDto(){
        System.out.println(dto); //dto를 출력하고 다시 dto 생성
        dto = new MemberDto();
    }

}
```

만약 `MemberDto`를 인스턴스변수 `final`로 선언하고 , `checkDto()` 메소드에서 `dto`를 출력하고 다시 객체를 생성한다면 `java: cannot assign a value to final variable dto` 라는 에러가 뜬다. 즉 기본 자료형처럼 참조 자료형도 두 번 이상 값을 할당하거나 새로 생성자를 사용해 초기화할 수없다. 

```java
public class FinalReferenceType {

    final MemberDto dto = new MemberDto(); //인스턴스 변수를 final로 선언
    public static void main(String[] args) {
        FinalReferenceType referenceType = new FinalReferenceType();
        referenceType.checkDto();
    }

    public void checkDto(){
        System.out.println(dto); //dto를 출력하고 다시 dto 생성
        //dto = new MemberDto();
        dto.name = "gildong";
        System.out.println(dto);
    }
}
```

하지만 메소드를 변경하고 실행하면 컴파일이 잘 된다. 그 이유는 `MemberDto` 객체는 두번 이상 생성할 수없다. 하지만 객체안에 있는 필드들은 `final`로 선언된 것이 아니기때문에 제약이 없다. 따라서 클래스가 `Final`이라도 그 안의 인스턴스 변수나 클래스 변수가 `final`이 아니라는 것을 꼭 기억해야한다.