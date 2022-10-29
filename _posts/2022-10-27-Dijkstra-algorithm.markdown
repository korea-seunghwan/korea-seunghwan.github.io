---
layout: post
title: "Dijkstra algorithm"
date: 2022-10-27 00:00:00 +0900
tag: data structure
background: "/img/bg-post.jpg"
---

## 최단 경로 알고리즘

- 가장 짧은 경로를 찾는 알고리즘
    - 한 지점에서 다른 한 지점까지의 최단 경로
    - 한 지점에서 다른 모든 지점까지의 최단 경로
    - 모든 지점에서 다른 모든 지점까지의 최단 경로
- 각 지점은 그래프에서 노드로 표현
- 지점 간 연결된 도로는 그래프에서 간선으로 표현


### 다익스트라 최단 경로 알고리즘 개요

- 특정한 노드에서 출발하여 다른 모든 노드로 가는 최단 경로를 계산
- 다익스트라 최단 경로 알고리즘은 음의 간선이 없을 때 정상적으로 동작
- 그리디 알고리즘으로 분류
    - 매 상황에서 가장 비용이 적은 노드를 선택해 임의의 과정을 반복


### 다익스트라 최단 경로 알고리즘 동작과정

- 출발 노드 설정
- 최단 거리 테이블 초기화
- 방문하지 않은 노드 중에서 최단 거리가 가장 짧은 노드를 선택
- 해당 노드를 거쳐 다른 노드로 가는 비용을 계싼하여 최단 거리 테이블 갱신
- 위 과정 3번, 4번 반복


### 다익스트라 알고리즘의 특징

- 그리디 알고리즘 : 매 상황에서 방문하지 않은 가장 비용이 적은 노드를 선택해 임의의 과정을 반복
- 단계를 거치며 한 번 처리된 노드의 최단 거리는 고정되어 더 이상 바뀌지 않는다
    - 한 단계단 하나의 노드에 대한 최단 거리를 확실히 찾는 것으로 이해할 수 있음
- 다익스트라 알고리즘을 수행한 뒤에 테이블에 각 노드까지의 최단 거리 정보가 저장된다
    - 완벽한 형태의 최단 경로를 구하려면 소스코드에 추가적인 기능을 더 넣어야한다


### 다익스트라 알고리즘: 간단한 구현 방법

- 단계마다 방문하지 않은 노드 중에서 최단 거리가 가장 짧은 노드를 선택하기 위해 매 단계마다 1차원 테이블의 모든 원소를 확인한다

```python

    import sys
    input = sys.stdin.readline
    INF = int(1e9)              # 무한을 의미하는 값으로 10억을 설정

    # 노드의 개수, 간선의 개수를 입력받기
    n, m = map(int, input().split())

    # 시작 노드 번호를 입력받기
    start = int(input())

    # 각 노드에 연결되어 있는 노드에 대한 정보를 담는 리스트를 만들기
    graph = [[] for i in range(n+1)]

    # 방문한 적이 있는지 체크하는 목적의 리스트를 만들기
    visited = [False] * (n+1)

    # 최단 거리 테이블을 모두 무한으로 초기화
    distance = [INF] * (n+1)

    # 모든 간선 정보를 입력받기
    for _ in range(m):
        a, b, c = map(int, input().split())
        # a번 노드에서 b번 노드로 가는 비용이 c라는 의미
        graph[a].append((b, c))

    # 방문하지 않은 노드 중에서, 가장 최단 거리가 짧은 노드의 번호를 반환
    def get_smallest_node():
        min_value = INF
        index = 0       # 가장 최단 거리가 짧은 노드(인덱스)
        for i in range(1, n+1):
            if distance[i] < min_value and not visited[i]:
                min_value = distance[i]
                index = i
        return index

    def dijkstra(start):
        # 시작 노드에 대해서 초기화
        distance[start] = 0
        visited[start] = True

        for j in graph[start]:
            distance[j[0]] = j[1]

        # 시작 노드를 제외한 전체 n-1개의 노드에 대해 반복
        for i in range(n-1):
            # 현재 최단 거리가 가장 짧은 노드를 꺼내서, 방문 처리
            now = get_smallest_node()
            visited[now] = True
            # 현재 노드와 연결된 다른 노드를 확인
            for j in graph[now]:
                cost = distance[now] + j[1]
                
                # 현재 노드를 거쳐서 다른 노드로 이동하는 거리가 더 짧은 경우
                if cost < distance[j[0]]:
                    distance[j[0]] = cost

    # 다익스트라 알고리즘 수행
    dikstra(start)

    # 모든 노드로 가기 위한 최단 거리를 출력
    for i in range(1, n+1):
        # 도달할 수 없는 경우, 무한이라고 출력
        if distance[i] == INF:
            print('INFINITY')
        else:
            print(distance[i])

```

### 다익스트라 알고리즘: 간단한 구현 방법 성능 분석

- O(V)번에 걸쳐서 최단 거리가 가장 짧은 노드를 매번 선형 탐색
- 시간 복잡도는 O(v^2)
- 전체 노드의 개수가 5000개 이하라면 이 코드로 문제를 해결할 수 있다
    - 하지만 10000개를 넘어가는 문제라면 위의 코드로 풀면 효율성 테스트에 걸린다


### 우선순위 큐(Priority Queue)

- 우선순위가 가장 높은 데이터를 가장 먼저 삭제하는 자료구조
- 여러 개의 물건 데이터를 자료구조에 넣었다가 가치가 높은 물건 데이터부터 꺼내서 확인해야하는 경우에 우선순위 큐를 이용할 수 있다
- 보통 표준 라이브러리 형태로 지원한다


### 우선순위 큐를 구현하는 방법 : 힙(Heep)

- 우선순위 큐를 구현하기 위해 사용하는 자료구조 중 하나
- 최소 힙과 최대 힙 존재
- 다익스트라 최단 경로 알고리즘을 포함해 다양한 알고리즘에서 사용

### 힙 라이브러리 사용 예제: 최소 힙

```python

    import heapq

    # 오름차순 힙 정렬
    def heapsort(iterable):
        h = []
        result = []

        # 모든 원소를 차례대로 힙에 삽입
        for value in iterable:
            heapq.heappush(h, value)
        
        # 힙에 삽입된 모든 원소를 차례대로 꺼내어 담기
        for i in range(len(h)):
            result.append(heapq.heappop(h))

        return result

    result = heapsort([1, 3, 5, 7, 9, 2, 4, 6, 8, 0])
    print(result)

'''
    실행 결과

    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
'''

```

- python에서의 힙은 기본적으로 최소 힙을 제공하기 때문에 최대 힙을 사용하고 싶다면 넣어주는 데이터의 형태를 바꿔줘야 한다

```python

    import heapq

    # 내림차순 힙 정렬(Heap Sort)
    def heapsort(iterable):
        h = []
        result = []

        # 모든 원소를 차례대로 힙에 삽입
        for value in iterable:
            heapq.heappush(h, -value)

        # 힙에 삽입된 모든 원소를 차례대로 꺼내어 담기
        for i in range(len(h)):
            result.append(-heapq.heappop(h))
        return result

    result = heapsort([1, 3, 5, 7, 9, 2, 4, 6, 8, 0])
    print(result)

```

### 다익스트라 알고리즘: 개선된 구현 방법

- 단계마다 방문하지 않은 노드 중에서 최단 거리가 가장 짧은 노드를 선택하기 위해 힙 자료구조를 이용
- 다익스트라 알고리즘이 동작하는 기본 원리는 동일
    - 현재 가장 가까운 노드를 저장해 놓기 위해 힙 자료구조를 추가적으로 이용한다는 점이 다르다
    - 현재의 최단 거리가 가장 짧은 노드를 선택해야 하기 때문에 최소 힙을 사용


### 다익스트라 알고리즘: 개선된 구현 방법 (python)

```python

    import heapq
    import sys

    input = sys.stdin.readline
    INF = int(1e9)              # 무한을 의미하는 값으로 10억을 설정

    # 노드의 개수, 간선의 개수를 입력받기
    n, m = map(int, input().split())

    # 시작 노드 번호를 입력받기
    start = int(input())

    # 각 노드에 연결되어 있는 노드에 대한 정보를 담는 리스트를 만들기
    graph = [[] for i in range(n+1)]

    # 최단 거리 테이블을 모두 무한으로 초기화
    distance = [INF] * (n+1)

    # 모든 간선 정보를 입력받기
    for _ in range(m):
        a, b, c = map(int, input().split())

        # a번 노드에서 b번 노드로 가는 비용이 c라는 의미
        graph[a].append((b, c))

    def dijkstra(start):
        q = []

        # 시작 노드로 가기 위한 최단 경로는 0으로 설정하여, 큐에 삽입
        heapq.heappush(q, (0, start))
        diatance[start] = 0
        
        while q:
            # 가장 최단 거리가 짧은 노드에 대한 정보 꺼내기
            dist, now = heapq.heappop(q)

            # 현재 노드가 이미 처리된 적이 있는 노드라면 무시
            if distance[now] < dist:
                continue
            
            # 현재 노드와 연결된 다른 인접한 노드들을 확인
            for i in graph[now]:
                cost = dist + i[1]
                # 현재 노드를 거쳐서, 다른 노드로 이동하는 거리가 더 짧은 경우
                if cost < distance[i[0]]:
                    distance[i[0]] = cost
                    heapq.heappush(q, (cost, i[0]))

    # 다익스트라 알고리즘을 수행
    dijkstra(start)

    # 모든 노드로 가기 위한 최단 거리를 출력
    for i in range(1, n+1):
        if distance[i] == INF:
            print("INFINITY")
        else:
            print(distance[i])

```

- 힙 자료구조를 이용하는 다익스트라 알고리즘 시간 복잡도는 O(ElogV)
- 노드를 하나씩 꺼내서 검사하는 반복문은 노드의 개수 V 이상의 횟수로는 처리되지 않는다
    - 결과적으로 현재 우선순위 큐에서 꺼낸 노드와 연결된 다른 노드들을 확인하는 총 횟수는 최대 간선의 개수(E)만큼 연산이 수행될 수 있다
- 직관적으로 전체 과정은 E개의 원소를 우선순위 큐에 넣었다가 모두 빼내는 연산과 매우 유사하다


### 플로이드 워셜 알고리즘 개요

- 모든 노드에서 다른 모든 노드까지의 최단 경로를 모두 계산
- 플로이드 워셜 알고리즘은 다익스트라 알고리즘과 마찬가지로 단계별로 거쳐 가는 노드를 기준으로 알고리즘을 수행한다
    - 다만 매 단계마다 방문하지 않은 노드 중에 최단 거리를 갖는 과정이 필요하지 않다
- 2차원 테이블에 최단 거리 정보를 저장
- 다이나믹 프로그래밍 유형에 속한다

- 각 단계마다 특정한 노드 k를 거쳐 가는 경우를 확인
    - a에서 b로 가는 최단 거리보다 a에서 k를 거쳐 b로 가는 거리가 더 짧은지 검사


### 플로이드 워셜 알고리즘

```python

    INF = int(1e9)      # 무한

    # 노드의 개수 및 간선의 개수를 입력받기
    n = int(input())
    m = int(input())

    # 2차원 리스트를 만들고, 무한으로 초기화
    graph = [[INF] * (n+1) for _ in range(n+1)]

    # 자기 자신에서 자기 자신으로 가는 비용은 0으로 초기화
    for a in range(1, n+1):
        for b in range(1, n+1):
            if a == b:
                graph[a][b] = 0

    # 각 간선에 대한 정보를 입력 받아, 그 값으로 초기화
    for _ in range(m):
        # A에서 B로 가는 비용은 C라고 설정
        a, b, c = map(int, input().split())
        graph[a][b] = c

    # 점화식에 따라 플로이드 워셜 알고리즘을 수행
    for k in range(1, n+1):
        for a in range(1, n+1):
            for b in range(1, n+1):
                graph[a][b] = min(graph[a][b], graph[a][k] + graph[k][b])

    # 수행된 결과를 출력
    for a in range(1, n+1):
        for b in range(1, n+1):
            if graph[a][b] == INF:
                print("INFINITY")
            else:
                print(graph[a][b])

```

### 플로이드 워셜 알고리즘 성능 분석

- 노드의 개수가 N개일 때 알고리즘상으로 N번의 단계를 수행
    - 각 단계마다 O(N^2)의 연산을 통해 현재 노드를 거쳐 가는 모든 경로를 고려
- 따라서 플로이드 워셜 알고리즘의 총 시간 복잡도는 O(N^3)


### <문제> 전보

- 핵심 아이디어: 한 도시에서 다른 도시까지의 최단 거리 문제로 치환 가능
- N과 M의 범위가 충분히 크기 때문에 우선순위 큐를 활용한 다익스트라 알고리즘을 구현


답안 예시

```python

    import heapq
    import sys

    input = sys.stdin.readline
    INF = int(1e9)

    n, m, start = map(int, input().split())

    # 각 노드에 연결되어 있는 노드에 대한 정보를 담는 리스트를 만들기
    graph = [[] for i in range(n+1)]

    # 최단 거리 테이블을 모두 무한으로 초기화
    distance = [INF] * (n+1)

    # 모든 간선 정보를 입력받기
    for _ in range(m):
        x, y, z = map(int, input().split())
        # x번 노드에서 y번 노드로 가는 비용이 z라는 의미
        graph[x].append((y, z))

    def dijkstra(start):
        q = []
        # 시작 노드로 가기 위한 최단 거리는 0으로 설정하여, 큐에 삽입
        heapq.heappush(q, (0, start))
        distance[start] = 0

        while q:
            # 가장 최단 거리가 짧은 노드에 대한 정보를 꺼내기
            dist, now = heapq.heappop(q)
            if distance[now] < dist:
                continue
            
            # 현재 노드와 연결된 다른 인접한 노드들을 확인
            for i in graph[now]:
                cost = dist + i[1]
                # 현재 노드를 거쳐서, 다른 노드로 이동하는 거리가 더 짧은 경우
                if cost < distance[i[0]]:
                    distance[i[0]] = cost
                    heapq.heappush(q, (cost, i[0]))

    dijkstra(start)

    # 도달할 수 없는 노드의 개수
    count = 0

    # 도달할 수 있는 노드 중에서 가장 멀리 있는 노드와의 최단거리
    max_distance = 0
    for d in distance:
        # 도달할 수 있는 노드인 경우
        if d != INF:
            count += 1
            max_distance = max(max_distance, d)

    # 시작 노드는 제외해야 하므로 count - 1을 출력
    print(count-1, max_distance)

```

### <문제> 미래 도시

내가 짠 코드

```python

    n, m = map(int, input().split())

    graph = [[] for _ in range(m+1)]

    target = 0
    visit = 0

    for i in range(m+1):
        if i == m:
            x, k = map(int, input().split())
            target = x
            visit = k

        if i < m:
            a, b = map(int, input().split())
            graph[a].append(b)

    # print(graph)
    # print(start)
    # print(visit)

    import heapq

    INF = 1e9
    distance1 = [INF] * (m+1)
    distance2 = [INF] * (m+1)

    def dijkstra(start, distance):
        q = []
        heapq.heappush(q, (start, 0))
        
        while q:
            now, dist = heapq.heappop(q)

            if distance[now] < dist:
                continue

            for i in graph[now]:
                cost = dist + 1
                if cost < distance[i]:
                    distance[i] = cost
                    heapq.heappush(q, (i, cost))

    # start에서 visit까지 가는 거리 구하기
    dijkstra(1, distance1)
    d1 = distance1[visit]

    dijkstra(visit, distance2)
    d2 = distance2[target]

    print(d1 + d2)

```

- 핵심 아이디어: 전형적인 최단 거리 문제이므로 최단 거리 알고리즘을 이용해 해결한다
- 1번 노드에서 X까지의 최단 거리 + X에서 K까지의 최단 거리를 계산
- 플루이드 워셜로 해결이 가능하다

정답 코드

```python

INF = int(1e9)

# 노드의 개수 및 간선의 개수를 입력받기
n, m = map(int, input().split())

# 2차원 리스트를 만들고 모든 값을 무한으로 초기화
graph = [[INF] * (n+1) for _ in range(n+1)]

# 자기 자신에서 자기 자신으로 가는 비용은 0으로 초기화
for a in range(1, n+1):
    for b in range(1, n+1):
        if a == b:
            graph[a][b] = 0

# 각 간선에 대한 정보를 입력 받아, 그 값으로 초기화
for _ in range(m):
    # A와 B가 서로에게 가는 비용은 1이라고 설정
    a, b = map(int, input().split())
    graph[a][b] = 1
    graph[b][a] = 1

# 거쳐 갈 노드 X와 최종 목적지 노드 K를 입력받기
x, k = map(int, input().split)

# 점화식에 따라 플로이드 워셜 알고리즘을 수행
for k in range(1, n+1):
    for a in range(1, n+1):
        for b in range(1, n+1):
            graph[a][b] = min(graph[a][b], graph[a][k] + graph[k][b])

# 수행된 결과를 출력
distance = graph[1][k] + graph[k][x]

# 도달할 수 없는 경우 -1을 출력
if distance >= INF:
    print(-1)
else:
    print(distance)

```