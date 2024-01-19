탐
# 그래프 탐색 - BFS & DFS

# 1. 그래프 탐색(Graph Search)

그래프 탐색(Graph Search)을 알기전에 먼저 그래프를 알아야한다. 여기서 그래프란 정점과 **(node, vertex)와 간선(edge)으로 구성된 자료구조**이다. 트리도 그래프에 속한다.

![gs1](https://github.com/princenim/TIL/assets/59499600/124b7efc-9fc1-46fa-85ec-5cf84e7a8e18)


## 1.1 그래프의 종류

그래프에도 종류가 있는데 크게 3가지로 나뉜다.

![gs2](https://github.com/princenim/TIL/assets/59499600/490582ba-57c4-4b36-a1b3-ab636211a283)

1. 무방향 그래프 : 간선에 방향이 없음.
2. 방향 그래프 : 간선에 방향이 존재
3. 가중치 그래프 : 간선에 가중치가 존재

이 그래프라는 자료구조에서 **시작 정점이 주어지면 시작점에서 모든 정점을 한 번씩 방문하는 탐색방법이 그래프 탐색**이다. 예를 들면 특정 도시에서 다른 도시로 갈 수 있는지, 전자 회로에서 특정 단자와 단자가 서로 연결되어있는지 확인할때 사용할 수 있다.
이 그래프 탐색에는 대표적으로 **BFS(넓이 우선 탐색)** 와 **DFS(깊이 우선 탐색)** 가 있다. BFS와 DFS는 동시에 모든 경우의 수를 무식하게 탐색하는 완전탐색이기도한데 다시 말해 그래프에서 완전탐색하는 알고리즘이라고 볼 수 있다.
그리고 트리는 그래프의 한 종류이다.

## 1.2 그래프 구현

먼저 BFS와 DFS를 구현하기 전에 프로그래밍에서 그래프를 표현해야하는데 **인접행렬**과 **인접리스트**로 표현할 수 있다.

### 인접행렬

그래프의 정점을 **2차원 배열**로 만든것이다.

<img width="663" alt="gs3" src="https://github.com/princenim/TIL/assets/59499600/d58770cf-c98d-4172-bfb4-6c0b68e8f0eb">

정점 a, b를 잇는 간선이 있을 경우 행렬(a,b)에 1을 표현한다.
무방향 그래프의 경우에는 행렬(a,b), 행렬(b,a) 모두에 표시하며, (1,1)과 같은 같은 정점은 0으로 표현하면 된다.
만약에 가중치가 있는 그래프라면 가중치를 1 대신 표현하면 된다.

장점 : 2차원 배열에 모든 노드들의 간선 정보가 존재하므로 두 노드를 연결하는 간선을 조회할 때 O(1)시간복잡도를 가진다.
단점 : 간선의 수와 무관하게 항상 N^2 크기의 2차원 배열이 필요하므로 메모리 공간이 낭비된다.

### 인접리스트 (adgacency list)

인접 리스트는 각각의 정점에 인접한 정점들을 **연결리스트로** 표현한것이다.

![gs4](https://github.com/princenim/TIL/assets/59499600/b10f76b5-c843-4591-9a7d-f2bd66bca97b)

장점 : 존재하는 간선만 관리하므로 메모리 사용이 효율적이다.
단점 : 두 노드를 연결하는 간선을 조회하거나 노드의 차수를 알기 위해서는 노드의 인접 리스트를 탐색해야 하므로 노드의 차수만큼의 시간이 필요하다.

# 2. BFS 와 DFS

![gs5](https://github.com/princenim/TIL/assets/59499600/9e411b4d-4ed6-41cf-a4d6-7839af26aeda)

## 2.1 BFS (너비 우선 탐색, Breadth-First Search)

BFS는 **넓게** 탐색한다는 의미로 루트 노드에서 시작해서 인접한 노드를 먼저 탐색하는 방법이다. 선입선출(First in First out)인 **큐(Queue)** 자료구조를 이용하는 것이 정석이다.

만약 위와 같은 그래프가 존재한다면

1. 시작 노드 ‘1’를 먼저 큐에 삽입하고 **방문 처리**를한다. (여기서 방문처리를 하는 이유는 재방문하지 않기 위함이다.)
2. 큐에서 노드 ‘1’를 꺼내고,  노드 ‘1’의  인접 노드 중에서 방문하지 않은 노드 ‘2,3,4’를 모두 큐에 삽입하고 방문 처리한다.
3. 큐에서 노드 ‘2’를 꺼내고 ‘2’의 인접노드 ‘5’를 큐에 넣는다.
4. 큐에서 노드 ’3’을 꺼내고, ‘3’의 인접노드 ‘6,7’을 큐에 넣는다.
5. 이렇게 반복하다보면

1 → 2 → 3 → 4→ 5 → 6→ 7→ 8→ 9의 순서대로 방문함을 알 수 있다.

### Java Code

<img width="512" alt="gs7" src="https://github.com/princenim/TIL/assets/59499600/2fa93e8d-8070-4bae-8b62-5fed90b6e5cd">

위와 같은 그래프가 이진트리가 존재할때 BFS로 순회하여 출력하면 `1 2 3 4 5 6 7` 이 나와야한다.

```java
class Node {
    int data; //값 저장

    Node lt, rt; //왼쪽과 오른쪽 자식의 주소를 저장

    public Node(int val) { //생성자
        this.data = val;
        lt = rt = null;
}
```

```java
public class BFS {
    //BFS는 큐를 사용
    Node root; //인스턴스 변수는 Heap 영역에 생성됨

    public static void main(String[] args) {
        //주어진 Tree를 만들기
        BFS tree = new BFS();
        tree.root = new Node(1);
        tree.root.lt = new Node(2);
        tree.root.rt = new Node(3);
        tree.root.lt.lt = new Node(4);
        tree.root.lt.rt = new Node(5);
        tree.root.rt.lt = new Node(6);
        tree.root.rt.rt = new Node(7);

        tree.solve();
    }

    public void solve() {
        Queue<Node> Q = new LinkedList<>();
        Q.add(root);
        int level = 0;//레벨
        while (!Q.isEmpty()) {
            int len = Q.size();
            System.out.print(level + " : ");
            for (int i = 0; i < len; i++) {
                Node cur = Q.poll();
                System.out.print(cur.data + " ");
                if (cur.lt != null) { //꺼내고 넣고
                    Q.add(cur.lt);
                }

                if (cur.rt != null) {
                    Q.add(cur.rt);
                }
            }
            level++;
            System.out.println();
        }
    }
	}
}
```

<img width="711" alt="스크린샷 2024-01-17 오전 10 46 29" src="https://github.com/princenim/TIL/assets/59499600/29667cbc-c104-4850-a6ae-1a169e2eb38d">

### BFS의 특징

- 최단거리 경로를 찾는데 사용한다.
- BFS와 마찬가지로 노드를 방문했는지의 여부를 체크해야한다. 그렇지 않으면 무한루프에 빠질 위험이 있다.
- 방문한 노드들을 차례대로 저장한 후 꺼낼 수 있는 Queue의 자료구조를 이용한다.



## 2.2 DFS (깊이 우선 탐색, Depth-First Search)

DFS은 깊게 탐색한다는 의미로 루트 노트에서 시작해 최대한 깊숙히 들어가 확인한 뒤 다시 돌아가 다른 루트로 탐색하는 방식이다. 즉 다음 분기로 넘어가기 전에 해당 분기를 완벽하게 탐색한다.

### Java Code

<img width="512" alt="gs7" src="https://github.com/princenim/TIL/assets/59499600/2fa93e8d-8070-4bae-8b62-5fed90b6e5cd">

```agsl

```


### DFS의 특징