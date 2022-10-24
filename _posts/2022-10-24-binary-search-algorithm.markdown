---
layout: post
title: "binary search algorithm"
date: 2022-10-24 00:00:00 +0900
tag: data structure
background: "/img/bg-post.jpg"
---

## 이진 탐색 알고리즘

- 순차 탐색 : 리스트 안에 있는 특정한 데이터를 찾기 위해 앞에서부터 데이터를 하나씩 확인하는 방법
- 이진 탐색 : 정렬되어 있는 리스트에서 탐색 범위를 절반씩 좁혀가며 데이터를 탐색하는 방법
    - 이진 탐색은 시작점, 끝점, 중간점을 이용하여 탐색 범위를 설정

### 이진 탐색의 시간 복잡도

- 단계마다 탐색 범위를 2로 나누는 것과 동일하므로 연산 횟수는 log_2N에 비례한다
- 탐색 범위를 절반씩 줄이며, 시간 복잡도는 O(logN)을 보장

이진탐색 소스코드: 재귀적 구현(python)

```python

    def binary_search(array, target, start, end):
        if start > end:
            return None
        
        mid = (start + end) // 2
        
        # 찾은 경우 중간점 인덱스 반환
        if array[mid] == target:
            return mid

        # 중간점의 값보다 찾고자 하는 값이 작은 경우 왼쪽 확인
        elif array[mid] > target:
            return binary_search(array, target, start, mid-1)

        # 중간점의 값보다 찾고자 하는 값이 큰 경우 오른쪽 확인
        else:
            return binary_search(array, target, mid+1, end)

    # n(원소의 개수)과 target(찾고자 하는 값)을 입력 받기
    n, target = list(map(int, input().split()))

    # 전체 원소 입력 받기
    array = list(map(int, input().split()))

    # 이진 탐색 수행 결과 출력
    result = binary_search(array, target, 0, n-1)
    if result == None:
        print('원소가 존재하지 않습니다')
    else:
        print(result + 1)

```

### 파이썬 이진 탐색 라이브러리

- bisect_left(a, x) : 정렬된 순서를 유지하면서 배열 a에 x를 삽입할 가장 왼쪽 인덱스를 반환
- bisect_right(a, x) : 정렬된 순서를 유지하면서 배열 a에 x를 삽입할 가장 오른쪽 인덱스를 반환

```python

    from bisect import bisect_left, bisect_right

    a = [1,2,3,4,4,8]
    x = 4

    print(bisect_lect(a, x))
    print(bisect_right(a, x))

```

### 값이 특정 범위에 속하는 데이터 개수 구하기

```python

    from bisect import bisect_left, bisect_right

    # 값이 [left_value, right_value]인 데이터의 개수를 반환하는 함수
    def count_by_range(a, left_value, right_value):
        right_index = bisect_right(a, right_value)
        left_index = bisect_left(a, left_value)
        return right_index - left_index

    # 배열 선언
    a = [1,2,3,3,3,3,4,4,8,9]

    # 값이 4인 데이터 개수 출력
    print(count_by_range(a, 4, 4))

    # 값이 [-1, 3] 범위에 있는 데이터 개수 출력
    print(count_by_range(a, -1, 3))

```

### 파라메트릭 서치 (Parametric Search)

- 파라메트릭 서치란 최적화 문제를 결정 문제('예' 혹은 '아니오')ㄹ 바꾸어 해결하는 기법
    - 특정한 조건을 만족하는 가장 알맞은 값을 빠르게 찾는 최적화 문제
- 일반적으로 코딩 테스트에서 파라메트릭 서치 문제는 이진 탐색을 이용하여 해결할 수 있다

### <문제> 떡볶이 떡 만들기

내가 짠 코드

```python

    n, m = map(int, input().split())

    rice_cake_list = list(map(int, input().split()))

    rice_cake_list.sort()

    results = []

    for i in range(1, rice_cake_list[-1]):
        tmp_rice_cake = 0
        for rice_cake in rice_cake_list:
            if rice_cake <= i:
                tmp_rice_cake += 0
            else:
                tmp_rice_cake += (rice_cake - i)

        if tmp_rice_cake >= m:
            results.append(i)

    results.sort()
    print(results[-1])

```

<문제 해결 아이디어>

- 적절한 높이를 찾을 때까지 이진 탐색을 수행하여 높이 H를 반복해서 조정
- '현재 이높이로 자르면 조건을 만족할 수 있는가?'를 확인한 뒤에 조건의 만족 여부에 따라 탐색 범위를 좁혀 해결
- 절단기의 높이는 0부터 10억까지 정수
    - 이렇게 큰 탐색 범위를 보면 이진 탐색으로 접근해야함
- 이진 탐색 과정을 반복하면 답을 도출할 수 있다
- 중간점의 값은 시간이 지날수록 '최적화된 값'이기 때문에, 과정을 반복하면서 얻을 수 있는 떡의 길이 합이 필요한 길이보다 크거나 같을 때마다 중간점의 값을 기록

정담 코드

```python

    n, m = list(map(int, input().split()))

    array = list(map(int, input().split()))

    # 이진 탐색을 위한 시작점과 끝점 설정
    start = 0
    end = max(array)

    # 이진 탐색 수행 (반복)

    result = 0
    while(start <= end):
        total = 0
        mid = (start + end) // 2
        for x in array:
            # 잘랐을 때의 떡의 양 계산
            if x > mid:
                total += x - mid

        # 떡의 양이 부족한 경우 더 많이 자르기 (왼쪽 부분 탐색)
        if total < m:
            end = mid - 1
        # 떡의 양이 충분한 경우 덜 자르기 (오른쪽 부분 탐색)
        else:
            result = mid
            start = mid + 1

    print(result)

```

- 내가 풀이한 풀이는 효율성 테스트에 걸릴 확률이 99%이다
- 절반씩 잘라서 비교하고 절반의 값을 업데이트 하는 방법으로 하기 때문에 효율적이다

### <문제> 정렬된 배열에서 특정 수의 개수 구하기

내가 짠 코드

```python

    from bisect import bisect_left, bisect_right

    n, x = map(int, input().split())

    array = list(map(int, input().split()))

    result = bisect_right(array, x) - bisect_left(array, x)

    if result == 0:
        print(-1)
    else:
        print(result)

```

- 일반적인 선형 탐색(Linear Search)로는 시간 초과 판정을 받는다
- 하지만 데이터가 정렬되어 있기 때문에 이진 탐색을 수행할 수 있다
- 특정 값이 등장하는 첫번째 위치와 마지막 위치를 찾아 위치 차이를 계산하여 문제를 해결할 수 있다