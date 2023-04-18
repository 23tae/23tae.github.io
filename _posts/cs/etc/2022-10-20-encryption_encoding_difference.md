---
title: "Encryption과 Encoding의 차이점"
date: 2022-10-20
categories: [Computer Science, etc_cs]
tags: [til]
---

# Encryption (암호화)

![Untitled](/assets/img/cs/etc/encryption.png)

- 평문을 **암호문으로 변환**하는 과정
    - 평문 (plaintext) : 읽을 수 있는 데이터
    - 암호문 (ciphertext) : 읽을 수 없는 데이터
- 기본적으로 **데이터를 안전하게 보관**하기 위해 사용된다.
- 반대 개념은 **복호화(decryption)**
    - 사용자가 암호화 키와 암호화 알고리즘을 알아야만 평문으로 다시 변환할 수 있다.
- 예시) AES, RSA, blowfish
    - AES(Advanced Encryption Standard, 고급 암호화 표준) : 미국 표준 기술 연구소에 의해 제정된 암호화 방식
    - RSA : 공개키 암호시스템의 일종. 암호화 뿐 아니라 전자서명이 가능한 최초의 알고리즘.
        - 공개 키 암호 방식
            - 암호화와  복호화에 이용하는 키가 다른 방식.
            - 누구나 가질 수 있는 **공개키**와 특정 사람만이 가지는 **개인키**로 구성된다.
            - 개인키로 암호화 한 정보는 그 쌍이 되는 공개키로만 복호화가 가능하고, 공개키로 암호화한 정보는 그 쌍이 되는 개인키로만 복호화가 가능
    - blowfish : 키(key) 방식의 대칭형 블록 암호.
        - 블록 암호 : 기밀성 있는 정보를 정해진 블록 단위로 암호화 하는 대칭 키 암호 시스템.
            - 대칭 키 암호(symmetric-key algorithm) : 암호화와 복호하에 같은 암호 키를 쓰는 알고리즘

# Encoding (부호화)

![Untitled](/assets/img/cs/etc/encoding.png)

- 다른 타입의 시스템에서 쉽게 사용될 수 있도록 **데이터의 형식을 변환**하는 과정
- 데이터를 부호화하는데 사용되는 **알고리즘이 공개**되어 있다.
- 알고리즘을 사용해 쉽게 읽을 수 있는 형태로 되돌릴 수 있다.(decode)
- 예시) ASCII, UNICODE, URL 인코딩, Base64

# Ref.

[Difference Between Encryption and Encoding - GeeksforGeeks](https://www.geeksforgeeks.org/difference-between-encryption-and-encoding/)  
[블로피시 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/블로피시)  
[블록 암호 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/블록_암호)  
[대칭 키 암호 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/대칭_키_암호)  
[공개 키 암호 방식 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/공개_키_암호_방식)  
[공개키 암호 알고리즘 - 해시넷](http://wiki.hash.kr/index.php/공개키_암호_알고리즘)
