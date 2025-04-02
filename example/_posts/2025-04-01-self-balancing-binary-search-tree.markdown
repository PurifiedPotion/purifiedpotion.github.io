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
    x.right = y # x의 오른쪽 자식은 y가 된다
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
        node.left = insert(node.left, key) # 왼쪽 자식 노드로 다시 재귀
    else:
        node.right = insert(node.right, key) # key가 노드의 값보다 크다면 오른쪽 자식 노드로 다시 재귀

    # 2. 높이 업데이트
    node.height = 1 + max(get_height(node.left), get_height(node.right)) # 왼쪽 서브트리와 오른쪽 서브트리의 맥스값에 + 1

    # 3. 균형 인수 계산
    balance = get_balance(node)

    # 4. 회전으로 균형 잡기
    # LL
    if balance > 1 and key < node.left.key:
        return right_rotate(node) # 이때의 노드는 삽입된 노드의 부모 노드이다

    # RR
    if balance < -1 and key > node.right.key:
        return left_rotate(node) # 이때의 노드는 삽입된 노드의 부모 노드이다

    # LR
    if balance > 1 and key > node.left.key:
        node.left = left_rotate(node.left)
        return right_rotate(node) # 이때의 노드는 삽입된 노드의 부모 노드이다

    # RL
    if balance < -1 and key < node.right.key:
        node.right = right_rotate(node.right)
        return left_rotate(node) # 이때의 노드는 삽입된 노드의 부모 노드이다

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

4. 빨강 노드의 자식은 반드시 검정이어야 한다

5. 어떤 노드에서 리프까지 가는 모든 경로에는 같은 개수의 검정 노드가 있다. 이를 검정 높이(Black Height)라고 함

여기서 4번과 5번 규칙 덕에 트리가 지나치게 한쪽으로 치우치는 것을 방지해. 실제로 레드블랙 트리의 높이는 2*log(n)이하로 제한돼

### 레드블랙 트리의 주요 연산(삽입 & 삭제)

레드블랙 트리는 삽입/삭제 이후에 균형을 맞추기 위해 회전(Rotation)과 재색칠(Recoloring)을 사용해

#### 삽입

- 일반적인 BST 방식으로 노드를 삽입한다. 새 노드는 항상 ***빨강***으로 시작한다

- 삽입 후 규칙 위반이 있으면 색 변경 혹은 회전으로 균형을 맞춘다

- 조상의 색을 바꾸거나, 삼촌 노드가 빨강인지 검정인지에 따라 여러 경우로 나뉜다

#### 삭제

- BST 규칙에 따라 삭제(기본 구조는 일반 이진 탐색 트리와 동일)

- 삭제 후 레드블랙 트리 속성 위반이 발생할 수 있음

- 위반된 속성(주로 검정 높이 문제)을 fixup 과정을 통해 수정

### 레드블랙 트리의 회전(균형을 위한 핵심)

- **좌회전(Left Rotate)** : 노드가 오른쪽으로 치우쳤을 때 사용

- **우회전(Right Rotate)** : 노드가 왼쪽으로 치우쳤을 때 사용

이 회전 연산은 트리의 **BST 성질**은 유지하면서, 구조를 살짝 바꿔서 균형을 맞춰준다

### 레드블랙 트리 삽입 그림으로 이해하기

예시: 레드블랙 트리에 다음 값을 순서대로 삽입해보자

10 → 18 → 7 → 15 → 16
~~~css
      10(B)
~~~
#### Step 1: 10 삽입

- 트리가 비어 있으니 10을 루트로 만든다.

- 새 노드는 기본적으로 빨강이지만, 루트는 항상 검정이어야 한다.

~~~css
      10(B)
          \
         18(R)
~~~
#### Step 2: 18 삽입

- 일반 BST 방식으로 오른쪽에 삽입

- 새 노드는 빨강

~~~css
        10(B)
       /     \
    7(R)     18(R)
~~~
#### Step 3: 7 삽입

- 7은 10보다 작으니 왼쪽에 삽입.

- 역시 새 노드는 빨강.

~~~css
        10(B)
       /     \
    7(R)     18(R)
            /
         15(R)
~~~
#### Step 4: 15 삽입

- BST 순서대로 18의 왼쪽에 삽입됨 (18의 왼쪽 자식)

- 새 노드 15는 빨강

🚨 문제 발생!

- 15(R)의 부모 18(R)도 빨강이다 → Red-Red Violation! (규칙 4 위반)

#### 해결: 삼촌 노드 색 확인

- 15의 삼촌 노드는 7(R) (부모 18(R)의 형제)

- 삼촌도 빨강 → Recoloring 진행!

~~~css
색 바꾸기:
  - 18 → 검정
  - 7  → 검정
  - 10 → 빨강 (부모로 올라감)

        10(R)
       /     \
    7(B)     18(B)
            /
         15(R)
~~~

- ❗근데 10은 루트 → 루트는 항상 검정 → 다시 10 → 검정으로 바꿈

~~~css
       10(B)
       /     \
    7(B)     18(B)
            /
         15(R)
~~~

✔ 균형 복구 완료 ✅

~~~css
        10(B)
       /     \
    7(B)     18(B)
            /
         15(R)
              \
             16(R)
~~~

#### Step 5: 16 삽입

- 16은 10 < 18 < 16 → 18의 왼쪽 → 15의 오른쪽에 삽입

🚨 다시 Red-Red 발생

- 15(R) → 부모

- 16(R) → 자식 → Red-Red violation

#### 해결: 삼촌 노드 색 확인

- 16의 삼촌은 없음 (NIL → 검정)

- 삼촌이 검정이고, 노드가 부모의 오른쪽 자식인 경우 → 좌회전 후 우회전

~~~css
→ 15와 16의 관계 변경
        18(B)
       /
    16(R)
    /
 15(R)
~~~

#### 회전 + 색 변경, 먼저 좌회전(Left Rotate) on 15

~~~css
→ 16이 위로 올라감 + 16 → 검정 / 18 → 빨강
        10(B)
       /     \
    7(B)     16(B)
            /     \
         15(R)    18(R)
~~~

#### 회전 + 색 변경, 다음 우회전(Right Rotate) on 18 + 색 변경

✔ 최종 결과, 규칙 모두 만족 ✅

| 상황 | 조치 |
|:---:|:---:|
| 부모, 삼촌 모두 빨강 | 부모/삼촌 → 검정, 조부모 → 빨강 (Recolor) |
| 부모 빨강, 삼촌 검정, 삽입 노드 방향이 꺾여 있음 | 회전 후 재배치 (Rotate) |
| 삽입 노드가 부모와 일직선 | 한 번의 회전으로 조정 |

위 표의 회전 후 재배치와 한 번의 회전의 경우 아래 표를 참고하면 돼

| 방향 | Case 이름 | 회전 방식 |
|:---:|:---:|:---:|
| Left-Left (LL) | 부모 왼쪽, 자식 왼쪽 | Right Rotate |
| Right-Right (RR) | 부모 오른쪽, 자식 오른쪽 | Left Rotate |
| Left-Right (LR) | 부모 왼쪽, 자식 오른쪽 | Left Rotate → Right Rotate |
| Right-Left (RL) | 부모 오른쪽, 자식 왼쪽 | Right Rotate → Left Rotate |

### 레드블랙 트리 삭제 그림으로 이해하기

예시: 아래의 레드블랙 트리에서 삭제해보자

10 → 5 → 15 → 1 → 7 → 12 → 17
~~~css
         10(B)
        /     \
     5(R)     15(R)
    /   \     /    \
  1(B)  7(B) 12(B) 17(B)
~~~

#### Step 1: 1 삭제하기 (간단한 경우)

- 1(B)는 리프 노드 (자식 없음)

- 삭제 후 검정 노드가 사라짐 → 검정 높이 불균형 발생

#### 해결 전략: Double Black 해결

- "검정 노드를 삭제하면", 부모나 형제의 색상에 따라 다양한 케이스로 나뉘고, 회전과 색 변경을 조합해 해결해야 해

- 삭제하려는 노드가 검정이면, 검정 높이가 1 줄어들어 규칙 5를 위반해

- 이를 "Double Black" 상태로 표현하고, 다음과 같은 규칙으로 해결해

#### Double Black 해결 케이스 총정리

(이해하기 쉽게 비유하자면, 삽입은 "Red-Red"를 잡는 것이고, 삭제는 "Double Black"을 잡는 거야)

##### Case 1: 형제 S가 빨강
~~~css
         P(B)
        /
     x(DB)
        \
         S(R)
~~~

➡ 부모와 형제의 색을 바꾸고, 회전

- S → 검정

- P → 빨강

- 회전(P 기준)

➡ Double Black 문제는 그대로지만, 형제를 검정으로 만들어 다른 Case로 전환함

##### Case 2: 형제 S가 검정, 자식 둘 다 검정

➡ 형제를 빨강으로 칠하고, 부모에게 Double Black을 전가

- S → 빨강

- x의 Double Black → P로 전파

➡ 루트까지 가면 루트는 그냥 검정으로 고치고 끝

##### Case 3: 형제가 검정이고, 형제의 가까운 자식이 빨강

➡ 형제의 자식과 형제를 회전 + 색 변경

- S와 자식 사이 회전

- 이후 다시 Case 4로 전환

##### Case 4: 형제가 검정이고, 형제의 먼 자식이 빨강

➡ 부모와 형제의 색을 바꾸고, 회전 후 자식 → 검정

- 이 경우 Double Black 문제 해결!

##### 예제로 실제 트리 변경 보기

삭제 전
~~~css
         10(B)
        /     \
     5(R)     15(R)
    /   \     /    \
  1(B)  7(B) 12(B) 17(B)
~~~

🔥 1(B) 삭제

- 1은 리프 노드 (자식이 NIL = 검정)

- 검정 노드 삭제 → Double Black 발생 at NIL 자리

➡ 부모 5(R) / 형제 7(B)

- 형제 7은 검정이고 자식 NIL들 = 검정 → Case 2

👉 해결 과정

- 7 → 빨강

- 5 → Double Black 상태 전파

- 5는 빨강이므로 Double Black 해소 후 5 → 검정

삭제 후 트리
~~~css
         10(B)
        /     \
     5(B)     15(R)
        \     /    \
       7(R) 12(B) 17(B)
~~~

삭제 요약

| Case | 조건 | 조치 |
|:---:|:---:|:---:|
| Case 1 | 형제 빨강 | 부모-형제 색 바꾸고 회전 → 다른 Case로 전환 |
| Case 2 | 형제 검정 + 자식 둘 다 검정 | 형제 → 빨강, 부모로 전파 |
| Case 3 | 형제 검정 + 가까운 자식 빨강 | 회전 후 Case 4로 |
| Case 4 | 형제 검정 + 먼 자식 빨강 | 색 바꾸고 회전 → 문제 해결 |