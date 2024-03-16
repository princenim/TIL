# 📌 Error와 Exception의 차이

# 1. 에러와 예외

![exception`](https://github.com/princenim/TIL/assets/59499600/ead0e56c-3cc1-422d-93d4-7a60d48d6774)


먼저 크게 자바의 예외처리의 종류는 크게 2가지가 있다. `에러(Error)`와 `예외(Exception)`이다. 에러와 예외는 모두 Object 클래스를 상속받는 `Throwable` 클래스를 상속 받는다.

**차이점은 프로그램 밖에서 발생했는지 안에서 발생했는지에 대한 여부이다**. 더 정확히 말하면 에러는 `프로세스`에 영향을 주고, 예외는 `쓰레드`에만 영향을 준다. 또한 에러는 개발자가 예측하거나 처리할수없으며 예외는 개발자가 예측하고 대비할 수있다.

## 1.1 에러(Error)

에러는 자바 프로그램 밖에서 발생한 오류를 말한다. 대표적인 예가 서버의 디스크가 고장났다거나 메인보드가 고장난다거나 하는 등의 자바 프로그램이 제대로 동작하지 못하는 경우를 말한다. 따라서 이름이 `Error`로 끝나면 에러이다.

## 1.2 예외(Exception)

예외는 자바 프로그램 안에서 발생한 오류를 말한다. 개발자가 예측하고 대비할수있다. 그리고 예외는 2가지로 나뉜다.이때 ‘예외처리를 해야하는지의 여부’에 따라 예외 종류가 달라진다. 해야하면 `Checked Exception`, 안해도되면 `Unchecked Exception`이다.

### Checked Exception

예외처리가 필수이며 처리하지 않으면 컴파일되지 않는다. 즉, `try-catch`나 `throw`로 감싸야한다. `Runtime Exception` 이외에 있는 모든 예외를 말한다. 대표적인 예로 `IOException`, `SQLException`이 있다.

### Unchecked Exception

예외 처리가 필수가 아니다.  `Runtime Exception`을 상속하는 클래스를 모두 `Unchecked Exception`이라고한다.  컴파일 할때 예외가 발생하지 않고 런타임시에 예외가 발생할 가능성이 있으며, 그리고 컴파일시에 체크를 하지 않기때문에 `Unchecked Exception`이라고한다. 예외처리 여부를 명시적으로 하지 않아도된다. 객체 참조가 null인 상태에서 객체의 멤버나 메소드를 참조하려고 하는 `NullpointException`, 배열에서 존재하지 않은 인덱스에 존재하려고 할때 발생하는  `IndexOutOfBoundsException`이 있다.