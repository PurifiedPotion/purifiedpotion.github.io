---
layout: post
title:  "자기 균형 이진 탐색 트리(Self-Balancing Binary Search Tree)"
date:   2025-03-30
hide_last_modified: true
---

* toc  
{:toc .large-only}

AVL 트리 (Adelson-Velsky & Landis) 와 레드블랙 트리 같은 경우 이진 탐색 트리(Binary Search Tree, BST)의 일종으로, 자기 균형 이진 트리(Self-balancing BST) 중 하나야. 자기 균형 이진 트리가 필요한 이유는 예전에 내가 다뤘던 이진 검색트리에 대해서 생각해 봐야해. 이전 이진 검색 트리 같은 경우 키의 오름차순으로 노드가 삽입되면 트리의 높이가 깊어져 선형 리스트가 되면 빠른 검색을 못해. 그래서 높이 제한을 하고자 균형 이진 트리가 고안되었어

![균형 이진 탐색 트리](/assets/img/blog/computerscience/균형이진탐색트리.png)

- AVL 트리는 모든 노드에 대해 ***왼쪽과 오른쪽 서브트리의 높이 차이(균형 인수)가 -1, 0, 1을 유지하도록 자동으로 균형***을 맞춰주는 트리야

- 레드 블랙 트리 같은 경우 ***조금 더 느슨하게 균형을 잡아서 삽입/삭제 시 연산이 더 빠르다는 장점***이 있어

## AVL 트리 (Adelson-Velsky & Landis)

일반적인 BST는 삽입/삭제가 반복되면 한쪽으로 치우쳐서 **선형 구조(Linked List)**가 되어버릴 수 있어. 그럼 검색 속도도 O(log n)이 아니라 O(n)이 되어버려 😥

그래서 AVL 트리는 이런 불균형을 방지해서 항상 O(log n) 시간복잡도를 보장하게 해주는 거지!

### AVL 트리의 핵심 요소

- **높이(height)** : 노드의 높이는 자신을 루트로 하는 서브트리의 최대 깊이를 의미해

- **균형 인수(balance factor)** : 아래의 값이 -1, 0, 1을 넘으면 균형이 깨졌다고 판단하고, 회전을 통해 다시 균형을 맞춰야해

~~~python
balance_factor = height(left subtree) - height(right subtree)
~~~

### AVL 작동 원리

균형 인수값이 넘어서면(트리의 균형이 깨지면), 4가지 유형의 회전 중 하나를 사용해서 균형을 맞춰

#### 1. LL회전(단순 오른쪽 회전)

- 왼쪽 자식의 왼쪽에 삽입된 경우

~~~python
    # LL
    if balance > 1 and key < node.left.key:
        return right_rotate(node)
~~~

#### 2. RR회전(단순 왼쪽 회전)

- 오른쪽 자식의 오른쪽에 삽입된 경우

~~~python
   # RR
    if balance < -1 and key > node.right.key:
        return left_rotate(node)
~~~

#### 3. LR회전(왼쪽 자식의 오른쪽 → 두 번 회전)

- 왼쪽 자식의 오른쪽에 삽입된 경우

~~~python
    # LR
    if balance > 1 and key > node.left.key:
        node.left = left_rotate(node.left)
        return right_rotate(node)
~~~

#### 4. RL회전(오른쪽 자식의 왼쪽 → 두 번 회전)

- 오른쪽 자식의 왼쪽에 삽입된 경우

~~~python
    # RL
    if balance < -1 and key < node.right.key:
        node.right = right_rotate(node.right)
        return left_rotate(node)
~~~

### AVL 함수

~~~python
class AVLNode:
    def __init__(self, key):
        self.key = key
        self.left = None
        self.right = None
        self.height = 1  # 노드 높이 초기값 (leaf = 1)

# 높이 반환 함수
def get_height(node):
    if not node:
        return 0
    return node.height

# 균형 인수 계산
def get_balance(node):
    if not node:
        return 0
    return get_height(node.left) - get_height(node.right)

def right_rotate(y):
    x = y.left # x는 y의 왼쪽 자식의 값을 받는다
    T2 = x.right # T2는 x의 오른쪽 자식의 값을 받는다

    # 회전 수행
    x.right = y
    y.left = T2

    # 높이 업데이트
    y.height = max(get_height(y.left), get_height(y.right)) + 1
    x.height = max(get_height(x.left), get_height(x.right)) + 1

    return x

def left_rotate(x):
    y = x.right
    T2 = y.left

    # 회전 수행
    y.left = x
    x.right = T2

    # 높이 업데이트
    x.height = max(get_height(x.left), get_height(x.right)) + 1
    y.height = max(get_height(y.left), get_height(y.right)) + 1

    return y

def insert(node, key):
    # 1. BST 삽입
    if not node:
        return AVLNode(key) # root 노드를 새로 생성성
    elif key < node.key: # key가 노드의 값보다 작다면
        node.left = insert(node.left, key) # 왼쪽노드로 다시 재귀
    else:
        node.right = insert(node.right, key) # key가 노드의 값보다 크다면 오른쪽 노드로 다시 재귀

    # 2. 높이 업데이트
    node.height = 1 + max(get_height(node.left), get_height(node.right)) # 왼쪽 서브트리와 오른쪽 서브트리의 맥스값에 + 1

    # 3. 균형 인수 계산
    balance = get_balance(node)

    # 4. 회전으로 균형 잡기
    # LL
    if balance > 1 and key < node.left.key:
        return right_rotate(node)

    # RR
    if balance < -1 and key > node.right.key:
        return left_rotate(node)

    # LR
    if balance > 1 and key > node.left.key:
        node.left = left_rotate(node.left)
        return right_rotate(node)

    # RL
    if balance < -1 and key < node.right.key:
        node.right = right_rotate(node.right)
        return left_rotate(node)

    return node
~~~

## 레드블랙 트리(Red-Black Tree)

- 트리의 균형을 유지해서 최악의 경우에도 탐색/삽입/삭제가 O(log n) 이 되게 만든다.

- 삽입/삭제 연산이 자주 일어나는 환경에 적합하다.

### 레드블랙 트리의 핵심 규칙

레드블랙 트리는 노드에 색상 정보를 추가해서 균형을 유지해.
각 노드는 빨간색(red) 또는 검은색(black) 중 하나야

1. 노드는 빨강 혹은 검정이다

2. 루트는 항상 검정이다

3. 모든 리프(NIL 노드)는 검정이다. 여기서 리프는 실제 값이 아니라 "없음(None)"을 나타내는 NIL 노드를 말해

4. 어떤 노드가 빨강이라면, 그 자식 둘은 반드시 검정이다. 즉, 빨강 노드는 연속으로 올 수 없다(Red-Red violation 방지)

5. 어떤 노드에서 리프까지 가는 모든 경로에는 같은 개수의 검정 노드가 있다. 이를 검정 높이(Black Height)라고 함

여기서 4번과 5번 규칙 덕에 트리가 지나치게 한쪽으로 치우치는 것을 방지해. 실제로 레드블랙 트리의 높이는 2*log(n)이하로 제한돼

### 레드블랙 트리의 주요 연산(삽입 & 삭제제)

레드블랙 트리는 삽입/삭제 이후에 균형을 맞추기 위해 회전(Rotation)과 재색칠(Recoloring)을 사용해

#### 삽입

- 일반적인 BST 방식으로 노드를 삽입한다. 새 노드는 항상 빨강으로 시작한다

- 삽입 후 규칙 위반이 있으면 색 변경 혹은 회전으로 균형을 맞춘다

- 조상의 색을 바꾸거나, 삼촌 노드가 빨강인지 검정인지에 따라 여러 경우로 나뉜다

#### 삭제

- 삭제는 조금 복잡한데, 삭제 후 검정 높이(black height)가 깨질 수 있기 때문에 여러 경우를 나눠 처리해

### 레드블랙 트리의 회전(균형을 위한 핵심)

- **좌회전(Left Rotate)** : 노드가 오른쪽으로 치우쳤을 때 사용

- **우회전(Right Rotate)** : 노드가 왼쪽으로 치우쳤을 때 사용

이 회전 연산은 트리의 **BST 성질**은 유지하면서, 구조를 살짝 바꿔서 균형을 맞춰준다

### 그림으로 이해하기

아래와 같은 트리가 있다고 해보자
~~~css
       10(B)
      /     \
   5(R)     20(R)
~~~



| 항목 | 설명명 |
|:---:|:---:|
| 크루스칼 | "전국 도로 목록을 정리해서, 가장 싸고 필요 없는 건 빼면서 도로 연결해 나가는 느낌" |
| 프림 | "서울에서 시작해서, 점점 가까운 도시를 하나씩 연결해서 전국으로 확장해 나가는 느낌" |