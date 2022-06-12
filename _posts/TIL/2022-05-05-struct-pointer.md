---
title: "구조체 포인터"
date: 2022-05-05T07:34:13.650Z
categories: [TIL]
tags: [c]
---
# 개요

**`struct 구조체이름 *포인터이름 = malloc(sizeof(struct 구조체이름));`**

구조체는 크기가 크기 때문에 효율성을 위해서는 포인터에 메모리를 할당해 사용함.

# 접근

구조체 멤버에 접근하는 방법

- 일반 변수로 선언한 경우
    - `.` 사용
        
        ```c
        struct Person p1;
        p1.age = 30;
        ```
        
- 포인터로 선언한 경우
    - `->` 사용
        
        ```c
        struct Person *p1 = malloc(sizeof(struct Person));
        p1->age = 30;
        ```
        
  - 구조체 포인터에서 `.` 으로 멤버 접근하는법
    
    ```c
    p1->age;
    (*p1).age;
    ```
    구조체 포인터를 역참조한 뒤 .으로 멤버에 접근
  
    `(*p1).age`와 같이 구조체 포인터를 역참조하면,    
    ~~pointer to~~ struct Person → struct Person
    


# Ref.
<https://dojang.io/mod/page/view.php?id=418>