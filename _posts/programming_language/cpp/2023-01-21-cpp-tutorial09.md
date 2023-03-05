---
title: "C++ 개념 정리 : STL (표준 템플릿 라이브러리)"
date: 2023-01-21
categories: [Programming Language, CPP]
tags: [cpp]
---

# STL (Standard Template Library)

- 주요 라이브러리
    - **컨테이너 (container)**
        - 임의의 타입의 객체를 보관
    - **반복자 (iterator)**
        - 컨테이너의 원소에 접근
    - **알고리즘 (algorithm)**
        - 반복자들을 통해 일련의 작업을 수행

# 컨테이너

- 종류
    - 시퀀스 컨테이너(sequence container)
    - 연관 컨테이너(associative container)
    - 컨테이너 어댑터(adapter container)

## 시퀀스 컨테이너

- 특징
    - 데이터를 선형으로 저장
    - 특별한 제약이나 규칙이 없음
    - 삽입된 요소의 순서가 그대로 유지됨.
- 종류
    - vector
    - deque
    - list
    - forward_list

### 벡터 (vector)

- 개념
    - 동적 배열의 클래스 템플릿 표현
    - 요소가 추가되거나 삭제될 때마다 자동으로 메모리를 재할당하여 크기를 동적으로 변경함
    - 임의 접근을 제공하는 가장 간단한 시퀀스 컨테이너
- 선언
    
    ```cpp
    vector<템플릿인수> 객체이름(생성자인수);
    ```
    
    - 템플릿 인수 : 벡터 컨테이너에 저장될 요소의 타입
    - 생성자 인수 : 벡터 컨테이너의 초기 크기. 생략하면 빈 벡터 생성.
- 복잡도
    - 임의의 위치 원소 접근 (`[], at`) : O(1)*O*(1)
    - 맨 뒤에 원소 추가 및 제거 (`push_back`/`pop_back`) : amortized O(1)*O*(1); (평균적으로 O(1)*O*(1) 이지만 최악의 경우 O(n)*O*(*n*) )
    - 임의의 위치 원소 추가 및 제거 (`insert, erase`) : O(n)*O*(*n*)

### 데큐 (deque)

- 개념
    - double-ended queue
    - 양쪽에 끝이 있는 큐(queue)
- 특징
    - 컨테이너의 양 끝에서 빠르게 요소를 삽입(enqueue)/삭제(dequeue)할 수 있음.
    - 원소들이 메모리 상에 연속적으로 위치하지 않는다.
        
        ![Untitled](/assets/img/programming_language/cpp/tutorial09/Untitled.png)
        
- queue와 deque의 차이
    - queue
        
        ![Untitled](/assets/img/programming_language/cpp/tutorial09/Untitled%201.png)
        
        - 데이터를 뒤에서만 삽입
        - 데이터를 앞에서 삭제
        - 데이터를 찾는 것은 맨 앞, 맨 뒤 모두 가능
    - deque
        
        ![Untitled](/assets/img/programming_language/cpp/tutorial09/Untitled%202.png)
        
        - 데이터를 앞, 뒤에서 삽입
        - 데이터를 앞, 뒤에서 삭제
- 사용 방법
    
    ```cpp
    deque<자료형 타입> 이름;
    ```
    

### 리스트 (list)

![Untitled](/assets/img/programming_language/cpp/tutorial09/Untitled%203.png)

- 개념
    - 양방향 연결 구조를 가진 자료형
- 특징
    - 임의의 위치에 있는 원소에 바로 **접근**할 수 없다.
    - 임의의 위치에 있는 **원소의 추가/삭제**가 O(1)으로 매우 빠르다.
    - 반복자가 **한 칸씩** 밖에 못 움직인다.
        - `itr + 3` 와 같은 이동은 불가능

### 시퀀스 컨테이너의 선택

- 벡터
    - 일반적인 상황
- 리스트
    - 맨 끝이 아닌 중간에 원소를 추가/제거할 일이 많다.
    - 원소들을 순차적으로만 접근한다.
- 덱
    - 맨 처음과 끝 모두에 원소를 추가할 일이 많다.

## 연관 컨테이너

- 특징
    - **키(key)**와 **값(value)**처럼 관련있는 데이터를 하나의 쌍으로 저장하는 컨테이너
    - 요소들에 대한 빠른 접근을 제공
    - 삽입되는 요소의 위치를 지정할 수 없음.
- 종류
    - set
    - multiset
    - map
    - multimap

### 집합 (set)

- 개념
    - 저장하는 데이터 그 자체를 키로 사용하는 컨테이너
    - 가장 단순한 연관 컨테이너
- 특징
    - 검색 속도가 매우 빠름
        - 오름차순으로 정렬된 위치에 요소를 삽입하기 때문
    - **키**의 **중복은 불가능**함.
- 멀티집합 (multiset)
    - 키의 **중복이 허용된** 집합
- 선언 방법
    
    ```cpp
    set<템플릿인수> 객체이름;
    ```
    

### 맵 (map)

- 개념
    - 키와 값의 쌍으로 데이터를 관리하는 연관 컨테이너
- 특징
    - 키의 중복은 불가능함
- 멀티맵 (multimap)
    - 키의 중복이 허용된 맵.
    - 하나의 키가 여러 개의 값과 연결됨
- 선언 방법
    
    ```cpp
    map<템플릿인수> 객체이름;
    ```
    

### 연관 컨테이너의 선택

- set
    - 데이터의 존재 유무만 궁금한 경우
- multiset
    - 중복 데이터를 허용하는 경우
- map
    - 데이터에 대응되는 데이터를 저장하려는 경우
- multimap
    - 중복 키를 허용하는 경우
- unordered_set, unordered_map
    - 속도가 매우 중요해서 최적화가 필요한 경우

# 반복자 (iterator)

- 개념
    - STL 컨테이너에 저장된 요소를 반복적으로 순회하여, 각각의 요소에 대한 접근을 제공하는 객체
    - 컨테이너에 저장된 **데이터를 순회**하는 과정을 일반화한 표현
        - 컨테이너의 구조나 요소의 타입과는 상관없다.
- 정의되어야 하는 연산자
    - 참조 연산자(`*`)
    - 대입, 관계 연산자
    - 증가 연산자(`++`)
- 종류
    - 입력 반복자(input iterator)
    - 출력 반복자(output iterator)
    - 순방향 반복자(forward iterator)
    - 양방향 반복자(bidirectional iterator)
    - 임의 접근 반복자(random access iterator)
- 주의사항
    - 컨테이너에 원소를 추가하거나 제거하게 되면 기존에 사용하였던 모든 반복자들을 사용할 수 없게 됨.
        
        ![Untitled](/assets/img/programming_language/cpp/tutorial09/Untitled%204.png)
        
    - 해결 방법
        - erase 함수에만 반복자를 만들어서 전달.
            
            ```cpp
            vec.erase(vec.begin() + i);
            ```

# 알고리즘

- 분류
    - 읽기 알고리즘(algorithm 헤더 파일)
    - 변경 알고리즘(algorithm 헤더 파일)
    - 정렬 알고리즘(algorithm 헤더 파일)
    - 수치 알고리즘(numeric 헤더 파일)
- 기타
    - algorithm, numeric 헤더 파일에 정의됨

## 읽기 알고리즘

- 특징
    - 컨테이너의 지정된 범위에서 특정 데이터를 읽기만 하는 함수
    - 컨테이너를 변경하지 않음
- 종류
    - find()
        - 두 개의 입력 반복자로 지정된 범위에서 특정 값을 가지는 첫 번째 요소를 가리키는 입력 반복자를 반환
    - for_each()
        - 두 개의 입력 반복자로 지정된 범위의 모든 요소를 함수 객체에 대입한 후, 대입한 함수 객체를 반환

## 변경 알고리즘

- 특징
    - 컨테이너의 지정된 범위에서 요소의 값만을 변경할 수 있는 함수
    - 컨테이너를 변경하지 않음
- 종류
    - copy()
        - 두 개의 입력 반복자로 지정된 범위의 모든 요소를 출력 반복자가 가리키는 위치에 복사
    - swap()
        - 두 개의 참조가 가리키는 위치의 값을 서로 교환
    - transform()
        - 두 개의 입력 반복자로 지정된 범위의 모든 요소를 함수 객체에 대입한 후, 출력 반복자가 가리키는 위치에 복사

## 정렬 알고리즘

- 특징
    - 컨테이너의 지정된 범위의 요소들이 정렬되도록 컨테이너를 변경하는 함수
    - 올바른 정렬을 위해 `임의 접근 반복자`를 사용
        - 임의 접근이 가능한 컨테이너만을 사용 가능
- 종류
    - sort()
        - 두 개의 임의 접근 반복자로 지정된 범위의 모든 요소를 서로 비교하여, 오름차순으로 정렬
    - stable_sort()
        - 두 개의 임의 접근 반복자로 지정된 범위의 모든 요소를 서로 비교하여, 값이 서로 같은 요소들의 상대적인 순서는 유지하면서 오름차순으로 정렬
    - binary_search()
        - sort() 함수를 사용하여 오름차순으로 정렬한 후에, 전달된 값과 같은 값이 있으면 참(true)을 반환하고 없으면 거짓(false)을 반환

## 수치 알고리즘

- algorithm 헤더 파일에 정의된 대부분의 알고리즘 함수와 다르게 **numeric**에 정의됨
- 주요 함수
    - accumulate()
        - 두 개의 입력 반복자로 지정된 범위의 모든 요소의 합을 반환함

# Ref.

[Difference between Queue and Deque (Queue vs. Deque) - GeeksforGeeks](https://www.geeksforgeeks.org/difference-between-queue-and-deque-queue-vs-deque/)
