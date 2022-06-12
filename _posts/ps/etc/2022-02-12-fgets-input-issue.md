---
title: "fgets함수가 입력을 받지 않음"
date: 2022-02-12
categories: [Problem Solving, etc_PS]
tags: [c, issue]
---
>scanf와 fgets를 동시에 사용할 때 fgets가 입력을 받지 않는 문제를 다룸

# Issue
여러 줄의 입력을 받을 때 앞에서 scanf()를 통해 입력을 받으면 뒤에 fgets()로 입력을 받지 않고 함수가 바로 종료되는 문제가 발생.


```c
#include <stdio.h>
#include <string.h>

struct Person {
    char name[20];
    int age;
    char address[100];
};

int main(void)
{
    struct Person p1;

    printf("이름을 입력하세요: ");
    scanf("%s",p1.name);
    printf("나이를 입력하세요: ");
    scanf("%d",&p1.age);
    printf("주소를 입력하세요: ");
    fgets(p1.address, 50, stdin);

    printf("이름: %s\n", p1.name);
    printf("나이: %d\n", p1.age);
    printf("주소: %s\n", p1.address); 
}
```
위 코드를 실행시키면 주소를 입력받을 것이라는 예상과 달리 아래처럼 주소를 입력하라고만 출력되고 바로 아래와 같이 결과가 출력된다.

---

```shell
이름을 입력하세요: 홍길동
나이를 입력하세요: 23
주소를 입력하세요: 이름: 홍길동
나이: 23
주소: 
```
---

# Cause
이 [링크](https://kldp.org/node/2754)에서 확인해보니 fgets 함수는 입력의 끝을 인식하는데 scanf에서 입력한 개행이 버퍼에 남아서 fgets가 끝으로 인식해 함수를 종료시킨다는 것이었다.
<br>

# Solution
위 문제와 관련하여 한 [블로그](https://wiserloner.tistory.com/418)에서 그 해결방법들을 제시해두었는데 본인은 scanf()와 fgets() 사이에 getchar()를 써서 남아있는 개행문자를 빼주었다. 그랬더니 그 후론 정상적으로 입력을 받게 되었다.

```c
#include <stdio.h>
#include <string.h>

struct Person {
    char name[20];
    int age;
    char address[100];
};

int main(void)
{
    struct Person p1;

    printf("이름을 입력하세요: ");
    scanf("%s",p1.name);
    printf("나이를 입력하세요: ");
    scanf("%d",&p1.age);
    getchar(); // 추가한 부분
    printf("주소를 입력하세요: ");
    fgets(p1.address, 50, stdin);

    printf("이름: %s\n", p1.name);
    printf("나이: %d\n", p1.age);
    printf("주소: %s\n", p1.address); 
}
```

위 코드를 실행한 결과는 아래와 같다


```shell
이름을 입력하세요: 홍길동
나이를 입력하세요: 23
주소를 입력하세요: 서울시 송파구 잠실동
이름: 홍길동
나이: 23
주소: 서울시 송파구 잠실동
```
