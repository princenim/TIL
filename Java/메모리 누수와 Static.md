# 1. 메모리 누수

## 1.1 메모리 누수(memory leak)란?

**자바의 메모리 누수는 더 이상 사용되지 않는 객체들이 GC에 의해 소멸되지 않고 누적되는 현상이다.** 따라서 어플리케이션 성능 저하를 일으키고, `Out Of Memory Error` 를 발생시키기 때문에 주의해야한다. 

그렇다면 언제 메모리 누수가 발생할까?

## 1.2 메모리 누수가 발생하는 경우

### 1. static 변수에 의한 객체 참조

결론부터 말하면 `static`을 사용했을 때 메모리 누수가 발생할 가능성이 있다. 왜냐하면 JVM의 `Rumtime data area`의 `Method area` ( = `static area`)는  GC의 관리 영역 밖에 있기 때문이다.

`static`은 ‘고정된’ 이라는 뜻으로 이 키워드를 사용해 **static 변수**와 **static 메소드, static 클래스** 를 만들 수있다.  `static` 은 객체에 소속되지않고, 클래스에 고정되어 있다고 생각하면 된다.

- s**tatic 변수  :** 인스턴스 변수에 static 붙은 변수를 말한다.
- **static 메소드**는 메소드에 static이 붙은 메소드를 말한다.
- **static 클래스**(= nested static class)는 `static`  가 붙어있는 내부 클래스를 말하며 , 외부클래스의 static 변수, static 메소드만 사용할 수 있다.  (`A class can be declared static only if it is a nested class.`)

여기서 확인해야할 점은 `static 변수`와 `static 클래스`는 JVM이 실행될 때 클래스 로더를 통해 바로 적재된다.

하지만 `static 메소드`는 클래스안에 `static 변수`가 존재할때 JVM이 실행될때 바로 적재되며

안에 `static 변수`가 없고,  아무것도 없을 때는 클래스를 사용하는 시점에 클래스 로더를 통해 적재된다.  *(이 부분 보충하기* )

이런 방식으로 **`Runtime data area`의 `Method area(= static area)`에 적재되었기 때문에 모든 쓰레드에서 객체를 공유할수 있는 수 있는 장점을 가지지만, 동시에 GC의 관리 밖에 존재한다.**

왜냐면 `Method area`는 JVM 구동 시작 시에 생성되고, 종료 시까지 유지되기 때문이다.
따라서 사용하지 않고 할당만 계속 한다면 GC가 발생하지 않아 메모리 누수로 이어지기 때문에 꼭 필요한 경우에만 static을 사용해야한다.

참고로 JVM이 실행 즉 구동될때는OS에서 메모리를 확보하며 , 이 메모리의 크기는 개발자가 JVM이  받을 메모리의 최소 , 최대를 설정가능하다.