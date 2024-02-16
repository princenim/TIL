# 📌 Priority Queue(우선순위 큐)와 힙(heap)

# **1. Priority Queue(우선순위큐)**

![pq](https://github.com/princenim/TIL/assets/59499600/a3844e6b-a5ed-42c3-8b28-4171c24f6190)

우선순위 큐는 일반적인 큐의 구조 `FIFO(First In First Out)`를 가지면서 데이터가 들어온 순서대로 나가는 것이 아닌 **우선순위가 높은 데이터가 먼저 나가는 자료구조**를 말한다.

따라서 큐에 저장할 객체들에게 우선순위가 필요하다.  우선순위를 정의하기 위해서는 `Comparable` 인터페이스를 구현하거나 `Comparator`을 구현하면 된다.  `Queue` 인터페이스를 상속받기때문에 `Queue` 에 정의된 메소드들을 사용할 수 있다.

우선순위 큐는 `힙(heap)`으로 구현하는 것이 일반적이다.

```java

//우선순위가 낮은 숫자가 먼저 나옴 (작은 숫자)
PriorityQueue<Integer> pQ = new PriorityQueue<>();

//우선순위가 높은 숫자가 먼저 나옴 (큰 숫자)
PriorityQueue<Integer> q = new PriorityQueue<>(Comparator.reverseOrder());  
```

## 1.1 **Comparable**

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
		    return this.age - o.age; //현재 객체 - 매개변수 객체 
    }

}
```

`Comparable`의 `compareTo()` 을 구현하면된다.

```java
public class CompareTest {
    public static void main(String[] args) {
        Student student1 = new Student(1, 2);
        Student student2 = new Student(2, 2);
        Student student3 = new Student(3, 2);

        PriorityQueue<Student> q = new PriorityQueue<>();
        q.add(student3);
        q.add(student2);
        q.add(student1);

        System.out.println(q);
        System.out.println(q.poll()); //값을 반환하고 제거
        System.out.println(q);
        System.out.println(q.poll());        
    }
}
```

```java
//결과 
[Student{age=1, classNumber=2}, Student{age=3, classNumber=2}, Student{age=2, classNumber=2}]
Student{age=1, classNumber=2}
[Student{age=2, classNumber=2}, Student{age=3, classNumber=2}]
Student{age=2, classNumber=2}
[Student{age=3, classNumber=2}]
```

해당 결과를 보면 `age`가 낮은 순서대로 출력됨을 알수 있다.

## 1.2 Comparator

```java
public class CompareTest {
    public static void main(String[] args) {
        Student student1 = new Student(1, 2);
        Student student2 = new Student(2, 2);
        Student student3 = new Student(3, 2);
        

        PriorityQueue<Student> q = new PriorityQueue<>(new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                return o1.age - o2.age;
            }
        });
        
        
        q.offer(student3);
        q.offer(student2);
        q.offer(student1);

        System.out.println(q);
        System.out.println(q.poll());
        System.out.println(q);
        System.out.println(q.poll());
        System.out.println(q);

    }
}
```

```java
//결과
[Student{age=1, classNumber=2}, Student{age=3, classNumber=2}, Student{age=2, classNumber=2}]
Student{age=1, classNumber=2}
[Student{age=2, classNumber=2}, Student{age=3, classNumber=2}]
Student{age=2, classNumber=2}
[Student{age=3, classNumber=2}]
```

결과를 보면 `age`가 낮은 순서대로 출력됨을 알수 있다.

# 2. 힙(heap)

**최소값 또는 최대값을 빠르게 찾아내기 위해 완전이진트리 형태로 이루어진 자료구조를 말한다.** 여기서 `이진트리`란 노드의 자식 노드가 2개개인 트리를 말라 `완전이진트리`란 이진트리에서 마지막 레벨을 제외하고 모든 레벨이 완전히 채워져 있으며 모든 노드가 왼쪽부터 채워져있는 트리를 말한다.

`우선순위 큐`는 `힙`을 사용해서 구현하는 것이 일반적이다. 배열과 연결리스트로도 구현을 할 수 있는데 왜 힙을 사용할까? `배열`으로 구현을 할 경우 삽입삭제는 빠르지만 삽입, 삭제 시 요소를 한칸씩 옮겨야하는 번거로움이 있고, `연결리스트`는 인덱스가 존재하지 않아서 우선순위를 탐색할때 처음부터 끝까지 탐색을 해야할 가능성이 있기때문에 성능을 저하시킨다. 따라서 힙을 사용해서 구현한다.

![heap](https://github.com/princenim/TIL/assets/59499600/1500f931-9ceb-4e13-a55b-f0a4c197deeb)

`힙`은 `최대힙`, `최소힙으로` 나눠진다.

- 최대힙 : 루트노드가 최대값이 되며 부모노드≥ 자식노드. 가장 큰 데이터가 우선적으로 제거되며
- 최소힙 :  루트노드가 최소값이 되며  부모노드 ≤ 자식노드

`힙`에서 원소의 삽입이나 삭제가 일어나면 최대 힙의 조건이 깨질 수 있다. 이 경우 다시 최대힙의 조건을 만족하도록 노드 위치를 바꾸는데 이를 `재구조화(heapify)`라고 한다.

### 삽입

![ㅁㅇㅇ](https://github.com/princenim/TIL/assets/59499600/eec5064b-7c94-4b8a-9e92-3342d6bfd48b)

18을 삽입한다고 가정하면 가장 말단 노드 즉 마지막 레벨의 비어있는 공간에 추가한다. 이때 힙 조건이 깨졌다. 따라서 `재구조화(heapify)`를 통해 속성을 맞춰줘야한다. 18의 부모노드인 5와 비교해 부모노드보다 클때 자리 교체를한다.  따라서  연산 자체는 `O(1)`의 시간복잡도를 가지지만 재구조화가 일어나기 떄문에 `O(logN)` 이다.

### 삭제

![delete](https://github.com/princenim/TIL/assets/59499600/977e0f9e-8620-47a5-a652-a0dc84a602e3)

힙의 최대값이 삭제되었다면 마지막 레벨의 마지막값인 5를 삭제된 요소의 위치에 옮긴다. 그리고 이 5가 올바른 자리에 위치할때까지 위에서 아래로 재구조화를 실행한다. 5는 16보다 작으므로 위치를 바꾼다. 그리고 5가 3보다 크고 오른쪽 자식노는 없으니까 둘의 위치를 바꾸지 않고 종료한다. 삭제의 경우 연산 자체는 `O(1)`의 시간복잡도를 가지지만 재구조화가 일어나기 떄문에 `O(logN)` 이다.

## 2.1 시간복잡도

시간복잡도는 다음과 같다.

- 삽입 :  연산 자체는 `O(1)`의 시간복잡도를 가지지만 재구조화가 일어나기 떄문에 `O(logN)` 이다.
- 삭제  : 연산 자체는 `O(1)`의 시간복잡도를 가지지만 재구조화가 일어나기 떄문에 `O(logN)` 이다.
- 읽기 : 최대/최소값을 기준으로 항상 트리 형태를 유지하기 때문에 `O(1)`이다.