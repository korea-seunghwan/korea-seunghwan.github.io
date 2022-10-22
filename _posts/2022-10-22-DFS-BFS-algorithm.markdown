---
layout: post
title: "DFS/BFS algorithm"
date: 2022-10-22 00:00:00 +0900
tag: data structure
background: "/img/bg-post.jpg"
---

## 그래프 탐색 알고리즘: DFS/BFS

- 탐색이란 많은 양의 데이터 중에서 원하는 데이터를 찾는 과정을 말한다
- 대표적인 그래프 탐색 알고리즘으로는 DFS와 BFS가 있다
- DFS/BFS는 코딩 테스트에서 매우 자주 등장하는 유형이므로 반드시 숙지 필요

### stack 자료구조

- 먼저 들어온 온 데이터가 나중에 나가는 형식(선입후출)의 자료구조
- python 에서는 list로 stack을 구현해서 사용할 수 있다 -> 시간복잡도가 O(1)이기 때문

### queue 자료구조

- 먼저 들어온 데이터가 먼저 나가는 형식(선입선출)의 자료구조
- 입구와 출구가 모두 뚫려있는 터널과 같은 형태로 시각화 할 수 있다
- python에서 구현할 때는

```python

    from collections import deque

    queue = deque()

```

로 사용해야 한다. list로 사용하면 시간복잡도가 다르기 때문에 효율성 체크에서 걸릴 수 있다.

### 재귀함수

- 자기 자신을 다시 호출하는 함수
- DFS를 구현하는데 실질적으로 자주 사용함으로 이해가 필수이다

```python

    def recursive_function():
        print('재귀 함수를 호출합니다')
        recursive_function()

    recursive_function()

```

- 재귀 함수를 문제 풀이에서 사용할 때는 재귀 함수의 종료 조건을 반드시 명시해야 한다
- 종료 조건을 제대로 명시하지 않으면 함수가 무한히 호출될 수 있다

### 재귀함수를 이용하는 몇가지 예제

- 팩토리얼 구현 예제

```python

    # 반복적으로 구현한 n!
    def factorial_iterative(n):
        result = 1
        # 1부터 n까지의 수를 차례대로 곱하기
        for i in range(1, n+1):
            result *= i
        return result

    # 재귀적으로 구현한 n!
    def factorial_recursive(n):
        if n <= 1:
            return 1
        # n! = n * (n-1)
        return n * factorial_recursive(n-1)

```

- 최대공약수 계산 (유클리드 호제법) 예제

    - 두 개의 자연수에 대한 최대공약수를 구하는 대표적인 알고리즘으로는 유클리드 호제법이 있다
    - 유클리드 호제법
        - 두 자연수 A, B에 대하여 (A>B) A를 B로 나눈 나머지를 R이라고 하자
        - 이때 A와 B의 최대공약수는 B와 R의 최대공약수와 같다
    - 유클리드 호제법의 아이디어를 그대로 재귀함수로 작성할 수 있다

```python

    def gcd(a, b):
        if a % b == 0:
            return b
        else:
            return gcd(b, a % b)

```

### DFS (Depth-First Search)

- DFS는 깊이 우선 탐색이라고도 부르며 그래프에서 깊은 부분을 우선적으로 탐색하는 알고리즘
- DFS는 스택 자료구조(혹은 재귀 함수)를 이용한다
    - 탐색 시작 노드를 스택에 넣고 방문 처리
    - 스택의 최상단 노드에 방문하지 않은 인접한 노드가 하나라도 있으면 그 노드를 스택에 넣고 방문 처리, 방문하지 않은 인접 노드가 없으면 스택에서 최상단 노드를 꺼냄
    - 더 이상 2번의 과정을 수행할 수 없을 때까지 반복

- DFS 소스코드 예제(Python)

```python

    # DFS 메서드 정의
    def dfs(graph, v, visited):
        # 현재 노드를 방문 처리
        visited[v] = True
        print(v, end=' ')
        # 현재 노드와 연결된 다른 노드를 재귀적으로 방문
        for i in graph[v]:
            if not visited[i]:
                dfs(graph, i, visited)

    # 각 노드가 연결된 정보를 표현 (2차원 리스트)
    graph = [
        [],
        [2,3,8],
        [1,7],
        [1,4,5],
        [3,5],
        [3,4],
        [7],
        [2,6,8],
        [1,7]
    ]

    # 각 노드가 방문된 정보를 표현 (1차원 리스트)
    visited = [False] * 9

    # 정의된 dfs 함수 호출
    dfs(graph, 1, visited)

```

### BFS (Breadth-First Search)

- BFS는 너비 우선 탐색이라고도 부르며, 그래프에서 가까운 노드부터 우선적으로 탐색하는 알고리즘
- BFS는 큐 자료구조를 이용한다
    - 탐색 시작 노드를 큐에 삽입하고 방문 처리
    - 큐에서 노드를 꺼낸 뒤에 해당 노드의 인접 노드 중에서 방문하지 않은 노드를 모두 큐에 삽입하고 방문처리
    - 더 이상 2번의 과정을 수행할 수 없을 때까지 반복

- BFS 소스코드 예제

```python

    from collections import deque

    # BFS 메서드 정의
    def bfs(graph, start, visited):
        # 큐(queue), 구현을 위해 deque 라이브러리 사용
        queue = deque([start])
        # 현재 노드를 방문 처리
        visited[start] = True
        # 큐가 빌때까지 반복
        while queue:
            # 큐에서 하나의 원소를 뽑아 출력하기
            v = queue.popleft()
            print(v, end=' ')
            # 아직 방문하지 않은 인접한 원소들을 큐에 삽입
            for i in graph[v]:
                if not visited[i]:
                    queue.append(i)
                    visited[i] = True

    # 각 노드가 연결된 정보를 표현 (2차원 리스트)
    graph = [
        [],
        [2,3,8],
        [1,7],
        [1,4,5],
        [3,5],
        [3,4],
        [7],
        [2,6,8],
        [1,7]
    ]

    # 각 노드가 방문된 정보를 표현 (1차원 리스트)
    visited = [False] * 9

    # 정의된 BFS 함수 호출
    bfs(graph, 1, visited)

```

### <문제> 음료수 얼려 먹기

- connected component 찾기
- 혼자 풀지 못했다.. 일단 답을 보며 이해를 해본다

- DFS를 활용한 알고리즘
    - 특정한 지점의 주변 상, 하, 좌, 우를 살펴본 뒤에 주변 지점 중에서 값이 '0'이면서 아직 방문하지 않은 지점이 있다면 해당 지점을 방문
    - 방문한 지점에서 다시 상, 하, 좌, 우를 살펴보면서 방문을 진행하는 과정을 반복하면, 연결된 모든 지점을 방문할 수 있다
    - 모든 노드에 대하여 1~2번의 과정을 반복하며, 방문하지 않은 지점의 수를 카운트

```python

    # DFS로 특정 노드를 방문하고 연결된 모든 노드들도 방문
    def dfs(x, y):
        # 주어진 범위를 벗어나는 경우에는 즉시 종료
        if x <= -1 or y <= -1 or x >= n or y >= m:
            return false
        
        # 현재 노드를 아직 방문하지 않았다면
        if graph[x][y] == 0:
            # 해당 노드 방문 처리
            graph[x][y] = 1
            # 상하좌우의 위치들도 모두 재귀적으로 호출
            dfs(x-1, y)
            dfs(x, y-1)
            dfs(x+1, y)
            dfs(x, y+1)
            return True
        return False

    # N, M을 공백을 기준으로 구분하여 입력 받기
    n, m = map(int, input().split())

    # 2차원 리스트의 맵 정보 입력 받기
    graph = []
    for i in range(n):
        graph.append(list(map(int, input())))

    # 모든 노드(위치)에 대하여 음료수 채우기
    result = 0
    for i in range(n):
        for j in range(m):
            # 현재 위치에서 DFS 수행
            if dfs(i, j) == True:
                result += 1

    print(result)

```

### <문제> 미로 탈출

내가 짠 코드

```python

    from collections import deque

    n, m = map(int, input().split())

    graph = []

    for i in range(n):
    graph.append(list(map(int, input())))

    queue = deque()
    queue.append((0,0,1))

    dx = [-1, 1, 0, 0]
    dy = [0, 0, -1, 1]
    result = 0

    while queue:
    x, y, count = queue.popleft()
    if x == n-1 and y == m-1:
        result = count
        break

    for i in range(len(dx)):
        next_dx = x + dx[i]
        next_dy = y + dy[i]

        if next_dx < 0 or next_dy < 0 or next_dx > n-1 or next_dy > m-1:
            continue

        if graph[next_dx][next_dy] == 1:
            queue.append((next_dx, next_dy, count+1))
            graph[next_dx][next_dy] = 0

    print(result)

```