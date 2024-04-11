
# 📌 그래프 탐색 - BFS & DFS

# 1. 그래프 탐색(Graph Search)

`그래프 탐색(Graph Search)`을 알기전에 먼저 그래프를 알아야한다. 여기서 그래프란 `정점 (node, vertex)`와 `간선(edge)`으로 구성된 자료구조이다. 트리도 그래프에 속한다.

![gs1](https://github.com/princenim/TIL/assets/59499600/124b7efc-9fc1-46fa-85ec-5cf84e7a8e18)


## 1.1 그래프의 종류

`그래프`에도 종류가 있는데 크게 3가지로 나뉜다.

![gs2](https://github.com/princenim/TIL/assets/59499600/490582ba-57c4-4b36-a1b3-ab636211a283)

1. **무방향 그래프** : 간선에 방향이 없음.
2. **방향 그래프** : 간선에 방향이 존재
3. **가중치 그래프** : 간선에 가중치가 존재

`그래프`라는 자료구조에서 **시작 정점이 주어지면 시작점에서 모든 정점을 한 번씩 방문하는 탐색방법이 그래프 탐색**이다. 예를 들면 특정 도시에서 다른 도시로 갈 수 있는지확인할때 사용할 수 있다 . A라는 도시(노드)에서 B라는 도시 (노드)도 이동하기 위해서는 간선(도로)를 거쳐서 이동할 수 있다. 또는 전자 회로에서 특정 단자와 단자가 서로 연결되어있는지 확인할때 사용할 수 있다.

이 그래프 탐색에는 대표적으로 `BFS(넓이 우선 탐색)`와 `DFS(깊이 우선 탐색)`가 있다. `BFS`와 `DFS`는 동시에 모든 경우의 수를 무식하게 탐색하는 `완전탐색`이기도한데 다시 말해 그래프에서 완전탐색하는 알고리즘이라고 볼 수 있다. 그리고 트리는 그래프의 한 종류이다.

## 1.2 그래프 구현

먼저 `BFS`와 `DFS`를 구현하기 전에 프로그래밍에서 그래프를 표현해야하는데 `인접행렬`과 `인접리스트`로 표현할 수 있다.

### 인접행렬

`2차원 배열`로 그래프의 연결관계를 표현하는 방식을 말한다.

<img width="663" alt="gs3" src="https://github.com/princenim/TIL/assets/59499600/d58770cf-c98d-4172-bfb4-6c0b68e8f0eb">

정점 a, b를 잇는 간선이 있을 경우 행렬(a,b)에 1을 표현한다. `무방향 그래프`의 경우에는 행렬(a,b), 행렬(b,a) 모두에 표시하며, (1,1)과 같은 같은 정점은 0으로 표현하면 된다. 만약에 가중치가 있는 그래프라면 가중치를 1 대신 표현하면 된다.

- 장점 : 2차원 배열에 모든 노드들의 간선 정보가 존재하므로 두 노드를 연결하는 간선을 조회할 때 O(1)시간복잡도를 가진다.
- 단점 : 간선의 수와 무관하게 항상 N^2 크기의 2차원 배열이 필요하므로 메모리 공간이 낭비된다.

### 인접리스트 (adjacency list)

`연결리스트로` 그래프의 연결관계를 표현하는 방식을 말한다. 인접리스트 방식은 모든 노드에 연결된 노드에 대한 정보를 차례대로 연결하여 저장한다.

![gs4](https://github.com/princenim/TIL/assets/59499600/b10f76b5-c843-4591-9a7d-f2bd66bca97b)

- 장점 : 존재하는 간선만 관리하므로 메모리 사용이 효율적이다.
- 단점 : 두 노드를 연결하는 간선을 조회하거나 노드의 차수를 알기 위해서는 노드의 인접 리스트를 탐색해야 하므로 노드의 차수만큼의 시간이 필요하다.

즉, 정리하면 메모리 측면에서 인접행렬방식은 모든 관계를 저장하므로 노드개수가 많을수록 불필요하게 낭비된다. 반면에 인접 리스트 방식은 연결된 정보만들 저장하기 때문에 메모리를 효율적으로 사용한다. 하지만 이와 같은 속성때문에 인접리스트 방식은 인접행렬 방식에 비해 특정한 두 노드가 연결되어있는지에 대한 정보를 얻는 속도가 느리다.


### 인접리스트 구현하기

- `가중치 없는 무방향 그래프`

<img width="764" alt="1" src="https://github.com/princenim/TIL/assets/59499600/d4c8f52d-44de-4e45-bf89-4fb058827a9b">

`가중치 없는 무방향 그래프`의 노드의 개수는 4개, 간선의 개수는 5개일때,

```java

    /*
    입력 예시 
    1 2
    1 3
    1 4
    2 4
    3 4
     */
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        int n = 4; //노드의 개수
        int m = 5; //간선의 개수

        // 인접 리스트 - ArrayList 안에 ArrayList
        ArrayList<ArrayList<Integer>> graph = new ArrayList<>();
        //초기화
        for (int i = 0; i<=n; i++){
            graph.add(new ArrayList<>());
        }

        //입력
        for(int i =0; i < m ; i++){
            String[] strArr=  br.readLine().split(" ");
            int n1 = Integer.parseInt(strArr[0]);
            int n2 = Integer.parseInt(strArr[1]);

            //(1 2)일 때 1번에 2번 연결 , 2번에 1번 연결
            graph.get(n1).add(n2);
            graph.get(n2).add(n1);
        }

        //출력
        for (int i = 1; i <= n; i++) {
            System.out.print("Node " + i + " is connected to: ");
            for (int j = 0; j < graph.get(i).size(); j++) {
                System.out.print(graph.get(i).get(j) + " ");
            }
            System.out.println();
        }

    }

//출력 결과
Node 1 is connected to: 2 3 4 
Node 2 is connected to: 1 4 
Node 3 is connected to: 1 4 
Node 4 is connected to: 1 2 3 
```

- `가중치 있는 방향 그래프`

<img width="752" alt="2" src="https://github.com/princenim/TIL/assets/59499600/b3bcc6ff-e1a8-44cf-afbd-87c8388c4d3a">


```java
public class GraphTest {

    /*
    입력 예시
    1 2 2
    1 3 2
    1 4 2
    2 4 2
    3 4 2
     */
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        int n = 4; //노드의 개수
        int m = 5; //간선의 개수

        // 인접 리스트 - ArrayList 안에 ArrayList
        ArrayList<ArrayList<Node>> graph = new ArrayList<>();
        //초기화
        for (int i = 0; i <= n; i++) {
            graph.add(new ArrayList<>());
        }

        //입력
        for (int i = 0; i < m; i++) {
            String[] strArr = br.readLine()
                                .split(" ");
            int from = Integer.parseInt(strArr[0]); //from 노드에서 to노드로 가중치만큼
            int to = Integer.parseInt(strArr[1]);
            int weight = Integer.parseInt(strArr[2]);

            graph.get(from)
                 .add(new Node(to, weight));
        }

        //출력
        for (int i = 0; i <= n; i++) {
            System.out.print("Node " + i + " is connected to: ");

            for (Node node : graph.get(i)) {
                System.out.print(node.to + "(Weight: " + node.weight + ") ");
            }
            System.out.println()
        }
    }

    static class Node {

        int to; // 도착 노드
        int weight; // 가중치

        Node(int to, int weight) {
            this.to = to;
            this.weight = weight;
        }
    }
}

//출력 결과 

Node 0 is connected to: 
Node 1 is connected to: 2(Weight: 2) 3(Weight: 3) 4(Weight: 2) 
Node 2 is connected to: 4(Weight: 4) 
Node 3 is connected to: 4(Weight: 5) 
Node 4 is connected to: 
```

# 2. BFS 와 DFS

![gs5](https://github.com/princenim/TIL/assets/59499600/9e411b4d-4ed6-41cf-a4d6-7839af26aeda)

## 2.1 BFS (너비 우선 탐색, Breadth-First Search)

`BFS`는 넓게 탐색한다는 의미로 루트 노드에서 시작해서 가까운 노드부터 먼저 탐색하는 방법이다. `선입선출(First in First out)`인 `큐(Queue)` 자료구조를 이용하는 것이 정석이다.

### 동작과정

1. 탐색 시작 노드를 큐에 삽입하고 방문 처리를한다.
2. 큐에서 노드를 꺼내 해당 노드의 인접 노드중에서 방문하지 않은 노드를 모두 큐에 삽입하고 방문 처리를하낟.
3. 위의 과정을 더 이상 수행할 수 없을때까지 반복한다.

![dfs](https://github.com/princenim/TIL/assets/59499600/235edf34-1b1a-42c4-9b5f-c8ecf53d47fc)

다음과 같은 그래프가 존재할때 `DFS`를 사용한 탐색 과정은 다음과 같다.

1. 시작 노드인 1을 큐에 삽입하고 방문처리를한다.
2. 큐에서 1을 꺼내고 방문 하지 않은 인접 노드 2,3,8을 모두 큐에 삽입하고 방문처리를한다.
3. 큐에서 노드 2를 꺼내고 방문하지 않은 인접노드 7을 큐에 삽입하고 방문처리를한다.
4. 큐에서 노드 3을 꺼내고 방문하지 않은 인접 노드 4,5를 모두 큐에 삽입하고 방문 처리를 한다.
5. 큐에서 노드 8을 꺼내고 방문하지  방문하지 않은 인접노드가 없으므로 무시한다.
6. 큐에서 노드 7을 꺼내고 방문하지 않은 인접 노드 6을 큐에 삽입하고 방문 처리를 한다.
7. 남아있는 노드에 방문하지 않은 인접 노드가 없다. 따라서 모든 노드를 차례대로 꺼낸다.

결과적으로 노드의 탐색 순서(큐에 들어간순서)는 `1 -> 2 -> 3 -> 8 -> 7 → 4 → 5 → 6`이다.

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
- 노드를 방문했는지의 여부를 체크해야한다. 그렇지 않으면 무한루프에 빠질 위험이 있다.
- 방문한 노드들을 차례대로 저장한 후 꺼낼 수 있는 `Queue`의 자료구조를 이용한다.



## 2.2 DFS (깊이 우선 탐색, Depth-First Search)

`DFS`은 깊게 탐색한다는 의미로 루트 노트에서 시작해 최대한 깊숙히 들어가 확인한 뒤 다시 돌아가 다른 루트로 탐색하는 방식이다. 즉 다음 분기로 넘어가기 전에 해당 분기를 완벽하게 탐색한다.
(한 루트에 연결된 노드를 최대한 깊이 탐색한다는 의미로 더 이상 진행할 수 없을때까지 계속해서 탐색을 진행한다.)

### 동작과정

1. 탐색 시작 노드를 스택에 삽입하고 방문처리를 한다.
2. 스택의 최상단 노드에 방문하지 않은 인접 노드가 있으면 그 인접 노드를 스택에 넣고 방문 처리를한다. 방문하지않은 인접 노드가 없으면 스택에서 최상단 노드를 꺼낸다.
3. 위의 과정을 더 이상 수행할 수 없을때까지 반복한다.

여기서 말하는 방문처리란 스택에 한번 삽입되어 처리된 노드가 다시 삽입되지 않게 체크하는 것을 의미한다. 방문처리를 함으로써 각 노드를 한번씩만 처리할 수 있다.

![dfs](https://github.com/princenim/TIL/assets/59499600/235edf34-1b1a-42c4-9b5f-c8ecf53d47fc)

다음과 같은 그래프가 존재할때 `DFS`를 사용한 탐색 과정은 다음과 같다.

1. 시작 노드인 1을 스택에 삽입하고 방문처리를한다.
2. 스택의 최상단 노드인 1에 방문하지 않은 인접 노드 2,3,8이 있다. 이중에서 가장 작은 노드 2를 스택에 넣고 방문처리한다.
3. 스택의 최상단 노드인 2에 방문하지 않은 인접 노드 7이 있다. 7을 스택에 넣고 방문처리한다.
4. 스택의 최상단 노드인 7에 방문하지 않은 인접 노드 6,8이 있다. 이중 가장 작은 노드 6을 스택에 넣는다.
5. 스택의 최상단 노드의 6에 인접노드가없다. 스택에서 6을 꺼낸다.
6. 스택에 최상단 노드인 7에 방문하지 않은 인접 노드 8이 있다. 8을 스택에 넣고 방문처리를 한다.
7. 이렇게 방문하지 않은 노드가 없을때까지 반복한다.

결과적으로 노드의 탐색 순서(스택에 들어간순서)는 `1→ 2 → 7 → 6 → 8 → 3 → 4 → 5` 가 된다.

`DFS`는 스택을 사용하는 알고리즘이기 때문에 실제 구현은 `재귀함수`를 사용했을때 간결하게 구현할 수 있다. 밑은 간결하게 구현한 예시이다.

### Java Code - 재귀를 이용한 DFS

위의 예제를 코드로 표현하면 다음과같다.

```java
public class DFSTest {

    /*
    입력
    8 9 1 
    1 2 
    2 7
    6 7
    7 8
    1 8
    1 3
    3 4
    3 5
    4 5
     */

    //방문처리할 배열
    static boolean[] visited;
    //그래프를 담을 연결 리스트
    static ArrayList<ArrayList<Integer>> graph = new ArrayList<>();

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine()); //문자열을 쪼개줌

        int n = Integer.parseInt(st.nextToken()); //정점
        int m = Integer.parseInt(st.nextToken()); //간선
        int v = Integer.parseInt(st.nextToken()); //시작 번호

        visited = new boolean[m];

        //1. 입력값을 통해 그래프를 만들기 - 무방향 인접리스트
        for (int i = 0; i <= n; i++) {
            graph.add(new ArrayList<>());
        }

        for (int i = 1; i <= m; i++) { //간선의 갯수만큼
            st = new StringTokenizer(br.readLine());
            int a = Integer.parseInt(st.nextToken());
            int b = Integer.parseInt(st.nextToken());
            graph.get(a)
                 .add(b);
            graph.get(b)
                 .add(a);
        }

        System.out.println(graph);

        dfs(v); //1번 노드부터 시작
    }

    public static void dfs(int x) {
        visited[x] = true; //방문처리 

        System.out.print(x + " ");
        for (int i = 0; i < graph.get(x).size(); i++) {
            int y = graph.get(x).get(i);
            if (!visited[y]) { //방문처리 안한 노드만
                dfs(y);
            }
        }
    }
}
```

### DFS의 특징
- 미로찾기와 같은 경로 문제에 유용하다.
- 노드를 방문했는지의 여부를 체크해야 한다. 이는 순환 구조에 빠지지 않도록 하는 역할을 한다.
- 깊은 단계의 노드를 빠르게 탐색할 수 있다. 하지만 최단 경로를 찾지 못할 수 있으며 무한루프에 빠질 가능성이 있다.
- 스택을 이용하여 구현 가능하며 재귀함수를 통해서 구현이 가능하다.