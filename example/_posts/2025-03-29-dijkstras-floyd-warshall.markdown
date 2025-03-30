---
layout: post
title:  "다익스트라(Dijkstra's)/플로이드-와샬(Floyd-Warshall)/벨만-포드 알고리즘(Bellman-Ford Algorithm) 알고리즘"
date:   2025-03-29
hide_last_modified: true
---

* toc  
{:toc .large-only}

파이썬 그래프에서 정점과 정점 사이의 거리를 구할때 쓰는 알고리즘 다익스트라(Dijkstra's), 플로이드-와샬(Floyd-Warshall), 벨만-포드 알고리즘(Bellman-Ford Algorithm) 알고리즘에 대해 설명할게. 세 알고리즘은 가중 방향 그래프가 기본이야

## 다익스트라 알고리즘 (Dijkstra's Algorithm) 기본

- 한 정점에서 다른 모든 정점까지의 최단 경로를 구할 때 사용해

- 간선의 가중치중에 음수가 없어야 함

- 시간 복잡도 : 기본 구현(O(V)^2) / 우선순위 큐(힙) 사용 시(O(V+E)logV)

먼저 시각화로 설명을 해볼게

### 다익스트라 알고리즘 시각화

~~~mathematica
   (2)
A ----> B
|       |
|       v
(5)     C
|       ^
v       |
D ----> E
   (1)
~~~

- 정점: A, B, C, D, E

- 간선: (A→B,2), (A→D,5), (B→C,1), (D→E,1), (E→C,2)

- 시작 정점: A

여기서 가장 짧은 경로를 계속 업데이트하면서, 가장 가까운 노드부터 처리하는 방식으로 접근해보자

~~~python
distance = {
    A: 0,
    B: ∞,
    C: ∞,
    D: ∞,
    E: ∞
}
~~~
먼저 A 기준에서 처음 초기 세팅을 해주고

~~~python
B 업데이트: 0 + 2 = 2
D 업데이트: 0 + 5 = 5
distance = {
    A: 0,
    B: 2,
    C: ∞,
    D: 5,
    E: ∞
}
~~~
A와 처음 만나는 정점 B와 D의 값을 업데이트 해준다

~~~python
C 업데이트: 2 + 1 = 3
distance = {
    A: 0,
    B: 2,
    C: 3,
    D: 5,
    E: ∞
}
~~~
그 다음에 B와 만나는 C를 업데이트 해준다

~~~python
E 업데이트: 5 + 1 = 6
distance = {
    A: 0,
    B: 2,
    C: 3,
    D: 5,
    E: 6
}
~~~
C에서 나가는 값이 없으므로 D기준으로 갈 수 있는 E값을 업데이트 해준다

E에서 C로 가는 값이 존재하지만, A → C로 가는 값이 더 낮기 때문에 업데이트 안해준다

그러면 여기서 A 기준 각 정점까지의 최단 거리가 아래 표처럼 나오게 된다

| 정점 | 최단거리 |
|:---:|:---:|
| A | 0 |
| B | 2 |
| C | 3 |
| D | 5 |
| E | 6 |

이거를 이제 재귀로 푸는 방식을 아래 보여줄게

### 다익스트라 알고리즘 (Dijkstra's Algorithm)

~~~python
import heapq

def dijkstra(graph, start): # 처음에 그래프와 시작하는 정점을 받는다
    distances = {node: float('inf') for node in graph} # distances라는 딕셔너리를 만들고 각 노드들에 무한대 값을 넣는다
    distances[start] = 0 # 시작하는 정점의 값은 무한대에서 0으로 바뀐다
    queue = [(0, start)] # 큐에 0과 start 튜플을 넣는다

    while queue: # 큐에 원소가 있으면
        current_dist, current_node = heapq.heappop(queue) # 큐에서 낮은 원소 팝한것을 current_dist, current_node에 저장한다. 처음 current_dist는 0

        if current_dist > distances[current_node]: # current_dist 가 distances[start 노드] 라면, 처음에는 같음. while 아래 for문 진행X
            continue                               # 해당 조건문이 있는 이유는 current_dist가 이미 더 크다면 최솟값을 구할 이유X

        for neighbor, weight in graph[current_node]: # 현재 노드값의 모든 이웃과, 가중치중 하나씩
            distance = current_dist + weight # distance는 current_dist + 가중치로 저장
            if distance < distances[neighbor]: # distance가 처음에는 무한대보다 작게된다
                distances[neighbor] = distance # 무한대값이 current_dist + 가중치로 update
                heapq.heappush(queue, (distance, neighbor)) # queue에 (distance,이웃) 튜플 push

    return distances
~~~

## 벨만-포드 알고리즘 (Bellman-Ford Algorithm) 기본



## 플로이드-와샬(Floyd-Warshall) 기본

- 모든 정점 쌍 사이의 최단 경로를 구할 때 사용

- 다익스트라와는 다르게, 음수 간선이 혀용됨(단, 음수 사이클은 없어야 함)

- ***음수 사이클***이란, 사이클을 돌고 났을때의 총 가중치가 음수가 되는 경로로써 계속 거리가 줄어들 수 있다. 그렇기 때문에, 이것과 비슷 한 ***벨만-포드*** 같은 최단 경로 알고리즘은 음수 사이클이 존재하면 안된다

- 시간 복잡도 O(V^3)

- 모든 도시 간의 최단 거리 미리 계산할때

다시 시각화로 설명을 해볼게

### 플로이드-와샬 알고리즘 시각화

~~~mathematica
    (3)
   A → B 
   ↑   ↓ (1)
 (8)   C
   ↑   ↓ (2)
   D ← E 
    (4)
~~~

- 정점: A, B, C, D, E

- 간선: (A→B,3), (B→C,1), (C→E,2), (E→D,4), (D→A,8)

모든 정점 쌍에 대해, 직접 가는 경로 vs 중간 노드를 거쳐 가는 경로를 비교하며 최단 거리 테이블을 업데이트 하는 방식으로 접근해 보자

~~~python
    A   B   C   D   E
A [ 0   3   ∞   ∞   ∞ ]
B [ ∞   0   1   ∞   ∞ ]
C [ ∞   ∞   0   ∞   2 ]
D [ 8   ∞   ∞   0   ∞ ]
E [ ∞   ∞   ∞   4   0 ]
~~~

자기 자신은 0, 나머지는 ∞로 초기 행렬을 먼저 설정한다.

~~~python
    A   B   C   D   E
A [ 0   3   4   10  6 ]
B [ ∞   0   1   ∞   ∞ ]
C [ ∞   ∞   0   ∞   2 ]
D [ 8   ∞   ∞   0   ∞ ]
E [ ∞   ∞   ∞   4   0 ]
~~~

A 자신을 중간 경유지로 고려로 시작하지만, 의미 없는것이어서 넘어가고

B를 중간 경유지로 고려했을때, A → C로 가는 경우가 있다. A → B + B → C = 3 + 1 = 4가 업데이트 됨

그렇다면 C를 경유하는 경우, A → E로 가는 경우가 있다. A → B + B → C + C → E = 3 + 1 + 2 = 6가 업데이트 됨

E를 경유하는 경우, A → D로 가는 경우가 있다. A → B + B → C + C → E + E → D = 3 + 1 + 2 + 4 = 10가 업데이트 됨

~~~python
    A   B   C   D   E
A [ 0   3   4   10  6 ]
B [ 12  0   1   7   3 ]
C [ 16  17  0   6   2 ]
D [ 8   11  12  0   14]
E [ 12  15  16  4   0 ]
~~~

위와 같은 방식으로 B, C, D, E 도 수행하면, 최종 결과가 도출돼

그러면 함수가 어떻게 짜여있는지 한번 보자

### 플로이드-와샬 알고리즘 (Floyd-Warshall Algorithm)

~~~python
def floyd_warshall(graph, n): # 처음에 그래프와 갯수를 받아온다
    dist = [[float('inf')] * n for _ in range(n)] # dist라는 리스트를 만들고 모든 값에 무한대를 저장한다
    
    for i in range(n): # 그래프의 갯수만큼
        dist[i][i] = 0 # dist 의 자기 자신끼리의 값에는 0으로 update한다
        
    for u, v, w in graph: # 그래프의 u:출발노드, v:도착노드, w:간선의 가중치 를 받아온다
        dist[u][v] = w # 경유 없이 가는 루트에 가중치를 대입한다

    for k in range(n): # 그래프의 갯수만큼
        for i in range(n): # 그래프의 갯수만큼 X 2 됐으니까 2차원 배열
            for j in range(n): # 그래프의 갯수만큼 X 3 됐으니까 3차원 배열, 바뀌는 속도는 k < i < j
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]) # dist[i][j]가 계속 변하는데, n만큼 반복된다. 대신에 k가 업데이트 되고 i에서 k + k에서 j 값과 min 비교
    
    return dist

graph = [
    (0, 1, 3),  # 0 → 1, 가중치 3
    (1, 2, 1),  # 1 → 2, 가중치 1
    (2, 4, 2),  # 2 → 4, 가중치 2
    (4, 3, 4),  # 4 → 3, 가중치 4
    (3, 0, 8)   # 3 → 0, 가중치 8
]
print(floyd_warshall(graph,len(graph)))
~~~

## 마무리

그러면 오늘 배웠던 다익스트라 알고리즘과 플로이드 와샬 알고리즘의 차이점을 알아보자

| 알고리즘 | 목적 | 입력 구조 | 시간복잡도 | 음수 가중치 | 음수 사이클 감지 |사용 상황 | 속도 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 다익스트라 | 한 정점 → 모든 정점의<br>최단 거리 | 인전 리스트 + 우선순위 큐 |O((V+E)log V) | 존재하면 안됨 | 감지 불가능 | 하나만 확인할 때 | 빠름 |
| 벨만-포드 | 한 정점 → 모든 정점의<br>최단 거리 | 간선 리스트 |O(V+E) | 존재해도 되지만,<br>음수 사이클은 존재하면 안됨 | 감지 가능 | 하나만 확인할 때,<br>음수 간선이 있을 때 | 느림 |
| 플로이드-와샬 | 모든 정점 쌍 사이 | 인접 행렬 |O(V^3) | 존재해도 되지만,<br>음수 사이클은 존재하면 안됨 | 감지 가능,dist[i][i]<0 |전체 거리를 계산할 때 | 느림 |


