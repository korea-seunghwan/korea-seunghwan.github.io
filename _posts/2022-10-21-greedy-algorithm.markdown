---
layout: post
title: "Greedy algorithm"
date: 2022-10-21 00:00:00 +0900
tag: data structure
background: "/img/bg-post.jpg"
---

## Greedy Algorithm
- 그리디 알고리즘(탐욕법)은 현재 상황에서 지금 당장 좋은 것만 고르는 방법을 의미
- 일반적인 그리디 알고리즘은 문제를 풀기 위한 최소한의 아이디어를 떠올릴 수 있는 능력을 요구
- 단순히 가장 좋아 보이는 것을 반복적으로 선택해도 최적의 해를 구할 수 있는지 검토

### <문제> 1이 될 때까지

내가 짠 코드

```python

    N, K = map(int, input().split())

    count = 0

    while N != 1:
        if N % K == 0:
            N = N // K
        else:
            N = N - 1

        count+=1

```


정답예시 코드

```python

    n, k = map(int, input().split())

    result = 0

    while True:
        # N이 K로 나누어 떨어지는 수가 될 때까지 빼기
        target = (n // k) * k
        result += (n - target)
        n = target
        # N이 K보다 작을 때 (더 이상 나눌 수 없을 때) 반복문 탈출
        if n < k:
            break

        # K로 나눠주기
        result += 1
        n /= k

    # 마지막으로 남은 수에 대하여 1씩 빼기
    result += (n-1)
    print(result)

```

내가 생각한 방법이랑은 다른 방법으로 예시를 풀었다. 일단 나눌 수 있는 수가 될때까지 (나눌 수 있는 가장 근접한 수)를 구해서 결과에 횟수로 더해주는 방법은 신선하게 다가왔다.

### <문제> 곱하기 혹은 더하기

내가 짠 코드

```python

    str_ints = map(int, input())
    result = 0

    for idx, str_int in enumerate(str_ints):
        if idx == 0:
            result = str_int
        
        if result == 0 or str_int == 0:
            result += str_int
        else:
            result *= str_int

    print(result)

```

정답예시 코드

```python

    data = input()

    # 첫 번째 문자를 숫자로 변경하여 대입
    result = int(data[0])

    for i in range(1, len(data)):
        # 두 수 중에서 하나라도 '0' 혹은 '1'인 경우, 곱하기 보다는 더하기 수행
        num = int(data[i])
        if num <= 1 or result <= 1:
            result += num
        else:
            result *= num

    print(result)

```

이번 문제는 크게 다르지 않은 것 같다. 핵심 포인트는 0 혹은 1일때는 더해주고 나머지는 다 곱해준다는 아이디어이다.

### <문제3> 모험가 길드

나혼자서는 못풀었다... 일단 시간이 없으니 얼른 해답을 보면서 그리디 사용법을 익히는데 초점을 둔다.

- 오름차순 정렬 이후에 공포도가 가장 낮은 모험가부터 하나씩 확인
- 앞에서부터 공포도를 하나씩 확인하며 '현재 그룹에 포함된 모험가의 수'가 '현재 확인하고 있는 공포도' 보다 크거나 같다면 이를 그룹으로 설정하면 된다.
- 이러한 방법을 이용하면 공포도가 오름차순으로 정렬되어 있다는 점에서, 항상 최소한의 모함가의 수만 포함하여 그룹을 결성하게 된다.

```python

    n = int(input())
    data = list(map(int, input().split()))
    data.sort()

    result = 0  # 총 그룹의 수
    count = 0   # 현재 그룹에 포함된 모험가의 수

    for i in data:          # 공포도를 낮은 것부터 하나씩 확인하며
        count += 1          # 현재 그룹에 해당 모험가를 포함시키기
        if count >= i:      # 현재 그룹에 포함된 모험가의 수가 현재의 공포도 이상이라면, 그룹 결성
            result += 1     # 총 그룹의 수 증가시키기
            count = 0       # 현재 그룹에 포함된 모험가의 수 초기화

    print(result)           # 총 그룹의 수 출력

```


'그룹'이라는 단어가 나오면 자연스럽게 set으로 만들어서 key값으로 하는 value값을 count하여 문제를 풀려고 하는 습관이 있으니 고치도록 하자.


## 구현: 시뮬레이션과 완전 탐색


- 구현이란, 머릿속에 있는 알고리즘을 소스코드로 바꾸는 과정
- 풀이를 떠올리는 것은 쉽지만 소스코드로 옮기기 어려운 문제들을 지칭
- 알고리즘은 간단한데 코드가 지나칠 만큼 길어지는 문제
- 실수 연산을 다루고, 특정 소수점 자리까지 출력해야 하는 문제
- 문자열을 특정한 기준에 따라서 끊어 처리해야 하는 문제
- 적절한 라이브러리를 찾아서 사용해야 하는 문제


### 시뮬레이션 및 완전 탐색 문제에서는 2차원 공간에서의 방향 벡터가 자주 활용된다.

```python

    # 동, 북, 서, 남    
    dx = [0, -1, 0, 1]
    dy = [1, 0, -1, 0]

    # 현재 위치
    x, y = 2, 2

    for i in range(4):
        # 다음 위치
        nx = x + dx[i]
        ny = y + dy[i]
        print(nx, ny)

```

어떤 위치에서 다른 위치로 방향성을 이용하여 이동시키는 일이 많다.


### <문제> 상하좌우

내가 짠 코드

```python

    ''' 입력 예시
    5
    R R R U D D
    '''

    n = int(input())
    direction_list = input().split()

    # 방향성 입력 R L D U
    direct = {'R':0, 'L':1, 'D':2, 'U':3}
    dx = [0, 0, 1, -1]
    dy = [1, -1, 0, 0]

    sx = 0
    sy = 0

    for direction in direction_list:
        next_directx = dx[direct[direction]]
        next_directy = dy[direct[direction]]

        next_dx = sx + next_directx
        next_dy = sy + next_directy

        if next_dx < 0 or next_dy < 0 or next_dx > n-1 or next_dy > n-1:
            continue

        sx = next_dx
        sy = next_dy

    print(sx+1, sy+1)

```


정답 풀이 코드
- 시뮬레이션 유형, 구현 유형, 완전 탐색 유형은 서로 유사한 점이 많다는 정도로 기억하면 된다.

```python

    # N 입력 받기
    n = int(input())
    x, y = 1, 1
    plans = input().split()

    # L, R, U, D에 따른 이동 방향
    dx = [0,0,-1,1]
    dy = [-1,1,0,0]
    move_types = ['L', 'R', 'U', 'D']

    # 이동 계획을 하나씩 확인하기
    for plan in plans:
        # 이동 후 좌표 구하기
        for i in range(len(move_types)):
            if plan == move_types[i]:
                nx = x + dx[i]
                ny = y + dy[i]
        
        # 공간을 벗어나는 경우 무시
        if nx < 1 or ny < 1 or nx > n or ny > n:
            continue
        
        # 이동 수행
        x, y = nx, ny

    print(x, y)

```

### 내가 오답이었던 이유
- 일단 n을 받을 때 int 처리를 안해줘서 정확히 연산이 안되었다
- 조건으로 matrix를 벗어났을 때 처리를 제대로 못해줬다.

### <문제> 시각
- 00시 00분 00초부터 N시 59분 59초까지의 모든 시각 중 3이 하나라도 포함되는 모든 경우의 수를 출력

```python

    n = int(input())

    count = 0
    for i in range(n+1):
        for j in range(60):
            for k in range(60):
                # 매 시각 안에 '3'이 포함되어 있다면 카운트 증가
                if '3' in str(i) + str(j) + str(k):
                    count += 1

    print(count)

```

- 여기서 풀이할 때 중요한점은 시,분,초에 대해서 datetime 개념으로 생각하는게 아니고 각각의 숫자에 대해서 59까지만 측정을 해주면 된다. -> 오히려 datetime 개념으로 생각하면 풀이가 어려워짐

### <문제> 왕실의 나이트

내가 짠 코드

```python

    # 8x8
    # [a,b,c,d,e,f,g,h]
    columns = 'abcedfgh'

    location = input()

    start_x = int(location[1])-1
    start_y = columns.find(location[0])

    dx = [-1, 1, -1, 1, -2, -2, 2, 2]
    dy = [-2, -2, 2, 2, -1, 1, -1, 1]

    count = 0
    for i in range(len(dx)):
        next_dx = start_x + dx[i]
        next_dy = start_y + dy[i]

        if next_dx < 0 or next_dy < 0 or next_dx > 7 or next_dy > 7:
            continue
        count += 1

    print(count)

```

- 나이트의 8가지 경로를 하나씩 확인하며 각 위치로 이동이 가능한지 확인
- 리스트를 이용하여 8가지 방향에 대한 방향 벡터를 정의

### <문제> 문자의 재정렬

내가 짠 코드

```python

    S = input()

    word_list = []
    num_list = []
    answer = ''

    for i in range(len(S)):
    s = S[i]
    if s.isdigit():
        num_list.append(int(s))
    else:
        word_list.append(s)

    word_list.sort()

    for word in word_list:
        answer += word

    answer += str(sum(num_list))
    print(answer)

```
- 그렇게 어려운 문제는 아니었다. 주어진 string을 slice로 하나씩 보면서 각각 모아주고 sorting과 sum을 통해 붙여줘야하는 문자를 구해서 정답을 도출해주는 문제