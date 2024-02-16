# 📌 Comparable과 Comparator의 차이점

`Comparable` 과 `Comparator` 인터페이스로 객체간의 비교를 하기 위해 만들어졌다. 그렇다면 두 인터페이스의 차이점은 무엇일까?
**결론부터 말하면 `Comparable`은 자기자신과 매개변수 객체를 비교하고, `Comparator`은 두 매개변수 객체를 비교한다.**

먼저 인터페이스이므로 구현 메소드를 확인해보자.

# 1. Comparable

[Java Development Kit Version 17 API Specification](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)

```java
public interface Comparable<T> {

    public int compareTo(T o);
}
```

`comparable`은 자기 자신과 매개변수 객체를 비교한다. `compareTo()` 메소드를 구현함으로써 자기자신과 매개변수 o의 객체를 비교할 수 있다.

```java
public class Student implements Comparable<Student> {
    int age;  //나이
    int classNumber; //학급

    Student(int age, int classNumber) {
        this.age = age;
        this.classNumber = classNumber;
    }

    //나이 기준으로 정렬
    @Override
    public int compareTo(@NotNull Student o) {
        //자기자신의 age가 더 클때 양수
        if (this.age > o.age) {
            return 1;
        } else if (this.age == o.age) {
            //같으면 0
            return 0;
        } else {
            //자기자신 age이 더 작을 때 음수
            return -1;
        }
    }
}
```

위의 예제는 나이(age)를 기준으로 비교를 하는 예이다. 따라서 자기 자신의 나이와 매개변수로 들어온 객체(o)의 나이값을 서로 비교한다. 그리고 `compareTo()`의 리턴값이 `int`임을 확인할 수 있다. 그리고 위의 `compareTo()` 메소드를 다음과 같이 수정할 수 있다.

```java
public class Student implements Comparable<Student> {
    int age;  //나이
    int classNumber; //학급

    Student(int age, int classNumber) {
        this.age = age;
        this.classNumber = classNumber;
    }

    @Override
    public int compareTo(@NotNull Student o) {
        return this.age - o.age;
    }
}
```

자기 자신의 age가 더 크다면 양수를 같면 0 , 작다면 음수를 반환할 것이다.

## 1.1  사용시 주의해야할점 - 오버플로우

위의 코드처럼 편하게 두 수의 차를 구해서 양수, 0, 음수를 구할수 있지만 이 방식에는 단점이 존재한다. 바로 뺄셈 과정에서 오버플로우가 발생할 수 있다. 해당 메소드는 `int`를 리턴한다. `int`의 범위는 `-2^31 ~ 2^31-1` 로  `-2,147,483,648 ~ 2,147,483,647`이다.

즉 만약 뺄셈 후의 값이 해당 범위를 넘어가면 문제가 발생한다. 예를들어 `this.age`가 `1`, `o,age`가 `-2,147,483,648` 일때 `1 - (-2,147,483,648)`= `2,147,483,649` 로 양수가 리턴되어야하는데  값이 오버플로우되어서 음수가 리턴된다. 하지만 음수가 나오면 자기자신이 더 작다는 상황이 되버린다.

따라서 두 수의 차를 이용해  `compareTo()`를 구현할때 오버플로우를 주의해야하고 `<, > , ==` 를 이용하는 방법이 조금더 안전하고 권장된다.

# 2. Comparator

[Java Development Kit Version 17 API Specification](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Comparator.html)

```java
public interface Comparator<T> {

    int compare(T o1, T o2);
}
```

`Comparator`은 두 매개변수 객체를 비교한다. `compare()` 를 오버라이드 함으로써의 매개변수로 들어온 두 객체를 비교할 수 있다. 즉 객체 자체와는 상관없이 매개변수로 넘어온 두 객체만들 비교한다.

```java
public class Student implements Comparator<Student> {
    int age;  //나이
    int classNumber; //학급

    Student(int age, int classNumber) {
        this.age = age;
        this.classNumber = classNumber;
    }

    @Override
    public int compare(Student o1, Student o2) { //매개변수로 들어온 두 객체를 비교
        if (o1.age > o2.age) {
            return 1;
        } else if (o1.age == o2.age) {
            return 0;
        } else {
            return -1;
        }

    }
}
```

위의 예제는 `compare()` 을 이용해 두 학생의 나이를 기준으로 비교하고 있다. 그리고 양수, 0 , 음수를 반환하고 있다.

```java
public class Student implements Comparator<Student> {
    int age;  //나이
    int classNumber; //학급

    Student(int age, int classNumber) {
        this.age = age;
        this.classNumber = classNumber;
    }
		
		@Override
		public int compare(Student o1, Student o2) {
	
			return o1.age - o2.age;
		}
}
```

만약 `o1`의 `age`가 `o2`의 `age`보다 크다면 양수가 반환 될 것이고 같다면 0을, 작다면 음수를 반환한다. 그리고 `Compaable` 처럼 값 차를 통해 값을 비교하면 오버플로우가 발생할 가능성이 있으므로 주의해야한다.