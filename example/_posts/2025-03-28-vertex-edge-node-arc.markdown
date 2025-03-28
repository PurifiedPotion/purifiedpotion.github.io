---
layout: post
title:  "그래프(Graph)"
date:   2025-03-28
hide_last_modified: true
---

* toc  
{:toc .large-only}

## 그래프(Graph) 자료구조에 사용되는 용어 

vertex, edge, node, arc에 대해 설명하고자 해

### Vertex(정점)

- 그래프에서의 '점', 즉 개체 하나하나를 의미함.

- 예: 사람, 도시, 웹페이지 등

- 예시 코드에서: 'A', 'B', 'C' 같은 것

- 📌 Node와 같은 뜻이야! (자세한 건 아래에서)

~~~python
graph = {
    'A': ['B', 'C'],
    'B': ['A'],
    'C': ['A']
}
~~~

여기서 'A', 'B', 'C'는 모두 vertex 또는 node라고 할 수 있어.

### Edge(간선)

- 두 vertex를 연결하는 선

- 예: 길, 친구 관계, 링크 등

- 양방향 혹은 단방향일 수 있음

~~~python
# A -- B
#  \
#   C

# 간선: A-B, A-C
~~~

- 위 코드에서는 ('A', 'B'), ('A', 'C')가 edge야.

- 양방향 그래프에서는 ('A', 'B') == ('B', 'A')

- 단방향 그래프에서는 다름 → 이게 아래 'Arc'와 연결됨

### Node(노드)

- 사실상 Vertex와 같은 의미

- 그래프 이론에서는 보통 vertex, 트리에서는 보통 node라고 함

- Node ≈ Vertex

### Arc(호)

- 단방향 간선, 방향이 있는 edge라고 생각하면 됨

- 예: A → B

~~~python
# 단방향 그래프
graph = {
    'A': ['B'],  # A → B
    'B': ['C'],  # B → C
    'C': []
}
~~~
여기서 (A, B)는 Arc, (B, A)는 없음.

### 그래프 용어 정리 마무리

그러면 각 용어에 대해 정리하면, 아래와 같아

| 용어 | 의미 | 비고 |
|:---:|:---:|:---:|
| Vertex | 그래프에서의 "정점" | Node와 같은 말 |
| Node | 구조에서의 "점" | Tree에서 사용하는 말 |
| Edge | 두 정점을 연결하는 선 | 양방향/단방향 모두 가능 |
| Arc | 방향이 있는 간선 | 단방향 edge |

파이썬에서 그래프 구현할 때는 ***딕셔너리 / 리스트 of 튜플/ 인접 행렬***을 보통 사용한다고 하니 참고하자


## 그래프 표현 방식

그래프란? 그래프는 ***노드(정점, Vertex)***와 ***간선(Edge)***로 구성된 자료구조야.

그래프의 표현 방식에 대해 알아볼게

### 인접 리스트 (Adjacency List)

가장 많이 쓰이는 방식으로 딕셔너리와 리스트로 구성돼 있어

~~~python
# 단방향 그래프
graph = {
    1: [2, 3],
    2: [4],
    3: [4],
    4: []
}
~~~

- 1은 2, 3과 연결됨

- 2는 4와 연결됨

- 연결된 노드를 리스트로 관리

### 인접 행렬 (Adjacency Matrix)

이차원 배열로 표현하는 방식이야

~~~python
graph = [
    [0, 1, 1, 0],  # node 0
    [0, 0, 0, 1],  # node 1
    [0, 0, 0, 1],  # node 2
    [0, 0, 0, 0],  # node 3
]
~~~

- graph[0][1] == 1 인데, 이 뜻은 0과 1이 연결되었다는 의미야

- 연결 여부만 보면 빠르지만, 메모리 소모 많음 (노드 수가 많으면 비효율적)

### 간선 리스트 (Edge List)

간선만 따로 리스트로 관리하는 방법이야

~~~python
edges = [
    (1, 2),
    (1, 3),
    (2, 4),
    (3, 4)
]
~~~

- 1은 2와 3과 연결되어 있다는 얘기야

- 보통 알고리즘 문제에서 그래프를 입력받을 때 자주 나와

- 이후 인접 리스트나 행렬로 변환해서 사용하는 경우 많음

## 방향 그래프 vs 무방향 그래프

여기서 방향 그래프(간선이 한 방향만 가능, 예 : 트위터 팔로우) 와 무방향 그래프(간선이 양방향, 예 : 친구 관계)의 차이점에 대해 다루고 갈게

***인접 리스트를 통해 비교해보자***

~~~python
graph = {
    1: [2],
    2: [],
}
~~~

위 아래를 비교해 보았을때, 위에는 1이 2로 가지만 2는 1로 가지 않으므로 ***방향 그래프***라고 볼 수 있고 
아래는 1에서 2로, 2에서 1로 가서 ***무방향 그래프***를 인접 리스트로 표현한것이지지

~~~python
graph = {
    1: [2],
    2: [1],
}
~~~

### 가중치 그래프 (Weighted Graph)

***서울에서 인천가는 버스***와 ***서울에서 부산가는 버스***의 ***가격***은 다르지? 그거를 표현할 수 있는게 가중치 그래프이고, ***가격외에도 거리, 시간에 대해 데이터***를 추가할 수 있어

~~~python
graph = {
    1: [(2, 4), (3, 1)],  # (노드, 비용)
    2: [(4, 2)],
    3: [(4, 5)],
    4: []
}
~~~

다익스트라(Dijkstra) 알고리즘에서 많이 사용한대

### 그래프 순회 (탐색)

BFS와 DFS를 통해 그래프 탐색도 가능한데, BFS와 DFS의 자세한 부분은 다음 블로그 포스팅을 참고하면 돼

~~~python
# DFS 예시
def dfs(graph, v, visited):
    visited[v] = True
    print(v, end=' ')
    for neighbor in graph[v]:
        if not visited[neighbor]:
            dfs(graph, neighbor, visited)

visited = {node: False for node in graph}
dfs(graph, 1, visited)
~~~

## 그래프 표현 방식 마무리

- 노드 번호가 0부터 시작하면 리스트로 표현

- 노드 번호가 1 이상이거나 알파벳이면 딕셔너리가 더 편해

- 파이썬에서는 아래와 같이 collections.defaultdict(list)를 쓰면 간편하게 그래프 만들 수 있어

~~~python
from collections import defaultdict

graph = defaultdict(list)
edges = [(1, 2), (1, 3), (2, 4)]
for u, v in edges:
    graph[u].append(v)
~~~

- 더 복잡한 그래프 연산이 필요하다면 networkx라는 라이브러리도 있으니 참고해

~~~bash
pip install networkx
~~~

↑가상환경에 설치

~~~python
import networkx as nx

G = nx.Graph()
G.add_edges_from([(1, 2), (1, 3)])
nx.shortest_path(G, 1, 3)
~~~

## 그래프의 연결 요소 찾기

### 연결 요소란?

- 무방향 그래프에서 모든 정점들이 경로로 연결된 하나의 묶음(예 : 친구 관계 그래프에서, 서로 연결된 친구들의 집단을 찾는 것)

- 그래프가 완전히 연결되어 있지 않다면, 여러 개의 연결 요소(묶음)가 존재할 수 있다

### 무방향 그래프에서의 연결 요소 찾기

예시 :
~~~python
graph = {
    1: [2],
    2: [1],
    3: [4],
    4: [3],
    5: []  # 혼자 떨어진 노드
}
~~~
위 그래프를 보면 {1, 2}, {3, 4}, {5} 로 연결 요소(묶음)이 3개가 나와

그렇다면 저 연결 요소를 어떻게 뽑아낼 수 있을까? DFS와 BFS 둘다 사용할 수 있어

예제 : DFS
~~~python
def dfs(graph, v, visited, component):
    visited[v] = True # 노드 방명록 싸인
    component.append(v) # 노드 리스트에 추가
    for neighbor in graph[v]: # 노드의 이웃들 하나씩
        if not visited[neighbor]: # 이웃 중에서 방문 안한 사람 중에서
            dfs(graph, neighbor, visited, component) # 방문 드가자

def find_connected_components(graph):
    visited = {node: False for node in graph} # 먼저 방명록 초기화
    components = [] # 연결 요소용 리스트

    for node in graph: # 그래프의 모든 노드 하나씩
        if not visited[node]: # 노드 하나씩인데, 방문 안한 노드중에서
            component = [] # 리스트 초기화
            dfs(graph, node, visited, component) # 그래프, 노드, 방명록, 리스트 송부
            components.append(component) # 이웃끼리의 리스트 component 추가
    
    return components

# 테스트
graph = {
    1: [2],
    2: [1],
    3: [4],
    4: [3],
    5: []
}

components = find_connected_components(graph)
print("연결 요소들:", components)
# 출력 결과 : [[1, 2], [3, 4], [5]]
~~~

재귀 사용

예제 : BFS
~~~python
from collections import deque

def bfs(graph, start, visited, component):
    queue = deque([start]) # start(노드)를 queue에 추가
    visited[start] = True # start(노드) 방명록 서명

    while queue: # 큐가 있다면
        v = queue.popleft() # 디큐해서 v로 저장 여기서는 start(노드)
        component.append(v) # 리스트에 start(노드) 추가
        for neighbor in graph[v]: # start(노드)의 모든 이웃중 한명씩
            if not visited[neighbor]: # 방문 안했다면
                visited[neighbor] = True # 방문 서명하고
                queue.append(neighbor) # 리스트에 추가

def find_connected_components_bfs(graph):
    visited = {node: False for node in graph} # 먼저 방명록 초기화
    components = [] # 연결 요소용 리스트

    for node in graph: # 그래프의 모든 노드 하나씩
        if not visited[node]: # 방문 안했다면
            component = [] # 리스트 초기화
            bfs(graph, node, visited, component) # graph, 노드, 방명록, 리스트 송부
            components.append(component) # 이웃끼리의 리스트 추가

    return components

# 테스트
graph = {
    1: [2],
    2: [1],
    3: [4],
    4: [3],
    5: []
}

print(find_connected_components_bfs(graph))
# 출력 결과 : [[1, 2], [3, 4], [5]]
~~~

큐 사용

#### 이렇게 구할 수가 있는데, DFS는 재귀 깊이 초과에 주의해야해. 만약 노드가 많다면 BFS를 사용하자

무방향 그래프에서의 연결 요소를 찾아 보았으니까. 이젠 방향 그래프에서의 강한 연결 요소를 찾아보자

### 방향 그래프에서의 강한 연결 요소 찾기

대표적으로 ***Tarjan 알고리즘***과 ***Kosaraju 알고리즘***이 있어

#### 강한 연결 요소(Strongly Connected Component)란?

- 방향 그래프에서, 모든 정점에서 서로 도달 가능한 최대 그룹을 얘기해

- 즉, 어떤 두 노드 u, v 가 있을 때, u → v 도 가고, v → u 도 갈 수 잇으면 → 같은 SCC

#### Tarjan 알고리즘 (DFS 기반, O(V + E))

하나의 DFS로 SCC를 찾는 고급 알고리즘으로 재귀적으로 방문하여, low-link 값을 이용해 SCC 판별

- DFS 도중 ***역방향 간선(back edge)***을 통해 자기 자신으로 돌아올 수 있는지를 체크

- stack을 활용해 SCC 구성 추적

~~~python
def tarjans_scc(graph):
    index = 0
    indices = {}
    lowlinks = {}
    stack = []
    on_stack = set()
    sccs = []

    def dfs(v):
        nonlocal index # nonlocal 같은 경우 외부 함수의 지역 변수 수정
        indices[v] = lowlinks[v] = index # indices[노드] 와 lowlinks[노드] 에 index(0) 저장
        index += 1 # 인덱스 1증가
        stack.append(v) # 스택에 노드 추가
        on_stack.add(v) # on_stack에다가도 노드 추가

        for w in graph[v]: # 노드의 이웃?
            if w not in indices: # 이웃이 indices에 없다면
                dfs(w) # 그 이웃 노드 dfs 수행
                lowlinks[v] = min(lowlinks[v], lowlinks[w]) # lowlinks[노드]에 노드의 이웃 또는 자신의 최솟값 저장
            elif w in on_stack: # 이웃이 on_stack에 있다면
                lowlinks[v] = min(lowlinks[v], indices[w]) # lowlinks[노드]에 lowlink[노드]와 indices[이웃]의 최솟값 저장

        # SCC 발견
        if lowlinks[v] == indices[v]: # lowlinks[노드] == indices[노드] 가 같다면
            scc = [] # 리스트 초기화
            while True: 
                w = stack.pop() # w 에 stack 팝된걸 저장
                on_stack.remove(w) # stack에서 w를 제거
                scc.append(w) # scc에 w 추가
                if w == v: # w 가 노드와 같다면
                    break
            sccs.append(scc) # w가 노드와 같지 않다면 sccs에 추가

    for node in graph: # 그래프의 모든 노드 하나씩
        if node not in indices: # indices 리스트에 없는 노드라면
            dfs(node) # 노드 데리고 dfs함수 실행

    return sccs

graph = {
    0: [1],
    1: [2],
    2: [0, 3],
    3: [4],
    4: []
}
print(tarjans_scc(graph))  # 출력 : [[4], [3], [0, 2, 1]]
~~~

#### Kosaraju 알고리즘 (두 번의 DFS, O(V + E))

SCC 찾는 데 있어서 가장 직관적인 방법으로 

- 정방향 DFS 후 → 노드 종료 순서 기록

- 그래프를 뒤집고(Transpose) 역방향 DFS 순회하여 SCC 추출

~~~python
from collections import defaultdict

def kosaraju_scc(graph):
    visited = set()
    stack = []

    # 1단계: 정방향 DFS로 종료 순서 기록
    def dfs1(v):
        visited.add(v)
        for w in graph[v]:
            if w not in visited:
                dfs1(w)
        stack.append(v)

    for node in graph:
        if node not in visited:
            dfs1(node)

    # 2단계: 그래프 뒤집기 (Transpose)
    reversed_graph = defaultdict(list)
    for u in graph:
        for v in graph[u]:
            reversed_graph[v].append(u)

    # 3단계: 종료 순서대로 역방향 DFS
    visited.clear()
    sccs = []

    def dfs2(v, component):
        visited.add(v)
        component.append(v)
        for w in reversed_graph[v]:
            if w not in visited:
                dfs2(w, component)

    while stack:
        v = stack.pop()
        if v not in visited:
            component = []
            dfs2(v, component)
            sccs.append(component)

    return sccs

graph = {
    0: [1],
    1: [2],
    2: [0, 3],
    3: [4],
    4: []
}
print(kosaraju_scc(graph))  # 예: [[4], [3], [0, 2, 1]]
~~~

Tarjan vs Kosaraju 차이점은 아래와 같아

| 항목 | Tarjan 알고리즘 | Kosaraju 알고리즘 |
|:---:|:---:|:---:|
| 방식 | DFS 1회, low-link 계산 | DFS 2회, 역방향 그래프 사용 |
| 시간복잡도 | O(V + E) | O(V + E) |
| 메모리 | 스택 필요 | 역방향 그래프 추가 필요 |
| 특징 | 한 번의 DFS로 끝남 | 더 직관적, 구현이 쉬움 |

방향 그래프에서 서로 연결된 컴포넌트를 묶고 싶을 때의 예는 아래와 같아

- 웹페이지 간 링크 순환 감지

- 모듈 간 순환 의존성 탐지

- 컴파일 의존성 정리

- 2-SAT 문제 해결의 핵심 알고리즘