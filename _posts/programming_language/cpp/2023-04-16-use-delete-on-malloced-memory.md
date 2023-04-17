---
title: "[C++] malloc으로 할당한 메모리를 delete로 해제해도 될까?"
date: 2023-04-16
categories: [Programming Language, CPP]
tags: [til, cpp, memory]
---

# 개요

C++은 로우 레벨의 메모리 관리와 하이 레벨의 추상화를 모두 지원하는 언어이다. C++의 주요 기능 중 하나인 메모리의 동적 할당, 해제는 `new`, `delete` 키워드를 통해 이루어진다. 이 연산자들은 생성자와 소멸자가 있는 **객체**를 사용하도록 설계되었으며 데이터 맞춤과 예외의 안전성을 처리한다.

반면 C는 생성자, 소멸자와 예외가 없는 로우 레벨 언어이다. C에서 메모리를 동적으로 할당, 해제하기 위해서는 `malloc()`과 `free()` 함수를 사용해야한다. 이런 함수들은 위 방식보다 더 **원시적**이기 때문에 할당된 메모리의 초기화나 메모리 정리를 수행하지 않는다. 또한 데이터 맞춤이나 에러 처리도 보장하지 않는다.

이 상황에서 다음과 같은 궁금증이 생길 수 있다. “C++의 `delete` 연산자를 사용하여 C의 `malloc()`으로 할당한 메모리를 해제해도 될까?” 결론부터 말하자면, **안된다**.

# 문제점

왜 안될까? 간단히 설명하면 `malloc()`과 `new`는 **heap의 서로 다른 공간에 할당**하기 때문이다. 이러한 작업은 **undefined behavior**(C/C++ 표준에서 어떤 결과가 나올 것인지를 명시하지 않은 작업)로, 컴파일러나 플랫폼에 따라 다른 결과가 나올 수 있다. `malloc()`된 메모리에 `delete`를 사용하여 나올 수 있는 문제로는 다음과 같은 것들이 있다.

- 메모리 누수(memory leaks)
    - `delete`가 `malloc()`으로 할당된 **메모리 블록을 찾거나 해제할 수 없어**서 메모리 자원을 낭비할 수 있다.
- 메모리 오염(memory corruption)
    - `delete`가 non-object에 대해 소멸자를 호출하거나 유효하지 않은 포인터에 접근하여 **crash**나 **데이터 유실**이 발생할 수 있다.
- 보안 취약점(security vulnerabilities)
    - `delete`가 일부 메모리에 예상할 수 없는 값을 덮어써서 **코드 인젝션**이나 권한 확대가 일어날 수 있다.
        - 코드 인젝션(code injection)
            - 공격자가 악의적인 코드를 사용자 입력 필드 애플리케이션에 주입하는 방법으로 공격하는 것
            - 이렇게 주입된 코드로 소프트웨어를 공격할 수도 있고 공격자의 권한을 확대(privilege escalation)하여 자신들이 원하는 데이터에 대한 접근 권한을 가질 수도 있다.

# 결론

이러한 문제들을 겪지 않으려면 **일치하는 쌍의 할당, 해제 함수**를 사용해야 한다. 만약 메모리를 `malloc()`으로 할당하였다면 해제는 `free()`를 통해 해야하고, `new`로 할당하였다면 해당 메모리는 `delete`로 해제되어야 한다. 또한 메모리가 사용자 정의 할당자(allocator)로 할당된 경우라면 해제 또한 일치하는 해제자(deallocator)로 해야한다.

# 참고

- [operator delete, operator delete[] - cppreference.com](https://en.cppreference.com/w/cpp/memory/new/operator_delete)
- [std::malloc - cppreference.com](https://en.cppreference.com/w/cpp/memory/c/malloc)
- [Behaviour of malloc with delete in C++ - Stack Overflow](https://stackoverflow.com/questions/10854210/behaviour-of-malloc-with-delete-in-c)
- [Memory Management, C++ FAQ](https://isocpp.org/wiki/faq/freestore-mgmt#mixing-malloc-and-delete)
- [c++ - What if, memory allocated using malloc is deleted using delete rather than free - Stack Overflow](https://stackoverflow.com/questions/20488282/what-if-memory-allocated-using-malloc-is-deleted-using-delete-rather-than-free)
- [Code injection - Wikipedia](https://en.wikipedia.org/wiki/Code_injection)
