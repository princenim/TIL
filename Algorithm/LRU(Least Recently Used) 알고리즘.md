# 📌 LRU(Least Recently Used) 캐시 알고리즘

# 1. 페이지 교체 알고리즘

페이지 교체는 컴퓨터의 가상 메모리 관리에서 중요한 개념이다. 일반적으로 운영체제는 물리적인 메모리(RAM)보다 큰 프로세스들을 실행할 수 있도록 가상 메모리를 제공한다. 이 가상 메모리는 페이지라고 불리는 고정 크기의 블록으로 나누어져있다.

페이지 교체란 현재 메모리에 적재되어있는 페이지들 중에서 어떤 페이지를 다른 페이지로 교체할지 결정하는 과정을 말한다. 메모리가 가득 차거나 새로운 페이지가 필요할때 발생한다. 이렇게 페이지를 교체할때 사용하는 알고리즘이 페이지 교체 알고리즘이다. 대표적으로 `FIFO(First In First Out)`, `LRU(Least Recently Used)`, `LFU(Least Frequently Used)`가 있다.

# 2. LRU(Least Recently Used) 알고리즘

`LRU(Least Recently Used)` 알고리즘은 **가장 최근에 사용되지 않은 데이터를 우선적으로 접근하는 제거하는 방법이다.**  LRU 알고리즘은 다양한 분야에서 사용되는데 운영체제의 페이지 교체 알고리즘, 데이터베이스의 버퍼관리, 웹 브라우저의 캐시관리 등에서 사용한다.

## 2.1 LRU 알고리즘 동작원리

1. `데이터 접근`
2. `접근 시간 갱신`
3. `가장 오래된 데이터 제거`
4. `새로운 데이터 추가`

다음과 같이 페이지 프레임 크기가 3일때 참조할 페이지가 `1,2,3,4,1,3,5,3,2,3`이라면 다음과 같은 순서대로 진행이 된다.

![123](https://github.com/princenim/TIL/assets/59499600/e502faa3-985f-487b-ae32-8d25beb5eb2e)


## 2.2 LRU 알고리즘 구현

그렇다면 위와 같은 알고리즘을 만들기 위해 어떤 자료구조를 사용해야할까?

(처음에 질문 받았을때 당당히 우선순위 큐라고 말하였다. 강렬하게 바로 틀려버림)

2가지 자료구조는 `Double Linked List(이중 연결 리스트)`와 `HashMap`이다. HashMap을 사용해서 데이터를 저장하고, `Double Linked List` 를 통해서 데이터의 접근 순서를 관리한다.

`Double Linked List`는 단순 `Linked List`와 다르게 양방향이다. 즉 노드에 이전노드와 다음노드의 주소가 같이 저장된다.

[LeetCode - LRU Cache](https://leetcode.com/problems/lru-cache/) 문제를 참고해서 구현을 해봤다.

```java
public class LRUCache {

    int capacity;//용량
    HashMap<Integer, Node> linkedList; //Key가 페이지 번호
    Node head, tail;

    public LRUCache(int capacity) { //생성자
        this.capacity = capacity;
        this.linkedList = new HashMap<>();
        this.head = new Node(0, 0);
        this.tail = new Node(0, 0);
        head.next = tail;
        tail.pre = head;
    }

    //맨 앞 노드 삭제 (제일 오래됐으니까)
    public int deleteNode() {
        Node delete = head.next; //삭제할 노드(맨 앞 노드)
        head.next = delete.next;
        delete.next.pre = head;
        return delete.key;
    }

    public int get(int key) {
        if (linkedList.containsKey(key)) {
            moveToTail(linkedList.get(key));
            return linkedList.get(key).val;
        } else {
            return -1;//없을떄
        }
    }

    public void put(int key, int value) {
        if (linkedList.containsKey(key)) {
            //키가 존재하면 tail로 이동
            linkedList.get(key).val = value;
            moveToTail(linkedList.get(key));
        } else {
            //키가 없을때 삭제하고 추가 후 꼬리 앞으로 이동, 그냥 추가 후 꼬리로 이동
            if (linkedList.size() == capacity) {//이미 찼으면 마지막꺼 삭제하고 추가
                linkedList.remove(deleteNode());
            }

            Node node = new Node(key, value);
            linkedList.put(key, node); //키가없으니까 새롭게 추가 (이떄 moveToTail 호출 시 새로 생겼기때문에 !=null 조건에 타지 않음)
            moveToTail(node);
        }
    }

    //꼬리 전으로 이동
    public void moveToTail(Node node) {
        if (node.pre != null) { //이전 노드가 있을때 먼저 내 자신을 삭제
            node.pre.next = node.next;
            node.next.pre = node.pre;
        }
        //나를 tail이전으로 이동
        Node dummy = tail.pre; //tail의 이전노드를 dummy라고 생각하기

        dummy.next = node;
        node.pre = dummy;
        tail.pre = node;
        node.next = tail;
    }

    class Node {

        int key;
        int val;
        Node pre;
        Node next;

        Node(int key, int val) {
            this.key = key;
            this.val = val;
            this.pre = null;
            this.next = null;
        }
    }
}
```

그림으로 표현하면 다음과 같다. 따라서 get도 `O(1)`의 시간복잡도, put도 `O(1)` 의 시간복잡도를 가진다.

![알고리즘-137](https://github.com/princenim/TIL/assets/59499600/7f54c8d9-9130-44e4-ab1c-a61ba76373c5)

두번째 그림은 `moveToTail` 을 표현한 것인데 새롭게 노드가 꼬리 전에 들어가려면 기존의 연결을 해제하고, 4개의 새로운 연결을 맺게된다. 이걸 코드에서 dummy를 이용해서 표현했는데 기존 tail의 이전 노드를 dummy라고 생각하고 dummy 기준으로 연결을 맺으면 편하다.