---
title: "C++ 개념 정리 : 메모리 동적 할당"
date: 2023-01-06
categories: [Programming Language, CPP]
tags: [cpp]
---

# 메모리 동적 할당

> new로 할당, delete로 해제

## 일반적인 경우
### 할당
    
```cpp
T* pointer = new T;
```
    
### 해제
    
```cpp
delete pointer;
```
    

## 배열에 할당하는 경우

### 할당

```cpp
T* arr = new T[size];
```
    
### 해제
    
```cpp
delete[] arr;
```