---
title:  "가비지컬렉션(Garbage Collection)의 종류와 특징"
subtitle: "동적 언어의 메모리 관리 기법"
tags:
    - computer science
date:   2021-12-26
---

가비지컬렉션 (garbage collection, GC)은 자동으로 메모리를 관리해주는 기법이다.
프로세스에 의해 할당되었지만, 더이상 참조되지 않는 메모리를 garbage라고 하고, 이러한 garbage를 수거하는 작업은 garbage collector가 진행한다.

java, python 등의 언어들은 이러한 GC를 염두에 두고 설계되어, 언어 자체에 해당 기능이 포함되어 있다.
- [cpython/gcmodule.c](https://github.com/python/cpython/blob/main/Modules/gcmodule.c)

C, C++ 등의 수동 메모리 관리를 가정하고 설계된 언어의 경우에도 GC를 지원하는 구현도 존재한다.

# 1. Tracing
tracing은 가장 대표적인 GC 기법이다.
tracing 알고리즘의 기본 개념은 root object로부터 접근 가능한 객체인지 추적한다는 것이다. 이러한 reference chaining 과정에서 탐지되지 않은 객체는 접근 불가능한 것이고, 따라서 GC의 대상이 된다.

## 기본 알고리즘 1-1. mark-and-sweep

이름 그대로 mark하고 sweep하는 알고리즘이다.
mark-and-sweep 알고리즘에서, 각각의 객체는 1비트의 flag를 하나씩 갖는다.

mark 단계에서는 root set 전체를 순회하며 flag를 '사용중' 상태로 설정한다.
sweep 단계에서는 메모리가 모두 스캔되었기에 '사용중' 상태가 아닌 메모리를 모두 free로 바꿀 수 있다.

이 기법은 단점이 몇가지 있는데, 그중 하나는 GC를 진행하는 동안 전체 시스템을 freeze 시켜야 한다는 점이다. 

![Animation_of_the_Naive_Mark_and_Sweep_Garbage_Collector_Algorithm](https://user-images.githubusercontent.com/34048253/147408809-ae9a3134-8b5b-4305-a30f-99fce81509bd.gif)

## 기본 알고리즘 1-2. Tri-color marking 

mark-and-sweep의 퍼포먼스 문제로 인해, 현대의 tracing GC는 tri-color marking 의 추상적 모델을 기반으로 다양하게 발전하고 있다.

tri-color marking은 아래와 같이 동작한다.

1. 각각의 객체를 흰색, 회색, 검은색으로 분류한다.
  - 흰색은 더이상 접근 불가능한 객체를 가리킨다.
  - 회색은 접근 가능한 객체이지만, 이 객체에서 가리키는 객체들은 아직 검사되지 않았음을 의미한다.
  - 검은색은 이 영역에서 가리키는 객체들이 흰색 객체를 가리키지 않음을 의미한다.
  - 알고리즘이 시작할 때는 변수가 가리키는 객체들이 회색으로 표시되며, 그 외의 모든 객체는 흰색으로 표시된다.
2. 회색으로 표시된 객체 가운데 하나를 선택하여 검은색으로 표시하고, 이 객체가 가리키는 모든 객체를 회색으로 표시한다.
3. 회색 객체가 하나도 남지 않을 때까지 위 과정을 반복한다.
4. 남은 흰색 객체는 접근 불가능한 객체이므로, 모두 해제한다.

이 알고리즘은 mark-and-sweep 알고리즘과는 달리, 상당한 시간동안 시스템의 중단 없이 즉시(on-the-fly) 수행이 가능하다.

![Animation_of_tri-color_garbage_collection](https://user-images.githubusercontent.com/34048253/147408817-0bff71e7-4528-431d-a127-f04b95326181.gif)

## 다양한 구현 전략
- Moving vs. non-moving
- Copying vs. mark-and-sweep vs. mark-and-don't-sweep
- Generational GC (ephemeral GC)
- Stop-the-world vs. incremental vs. concurrent
- Precise vs. conservative and internal pointers

# 2. Reference counting
레퍼런스 카운팅 방식의 GC에서 각 객체는 참조당하는 횟수를 표시해둔다.
이때 참조 횟수가 0이라면 Garbage라고 할 수 있다.
참고가 생기면 count는 증가하고, 참조가 사라지면 감소한다.
count가 0이 되었을때, 객체를 해제한다.

## update 비효율을 다루는 방법
작성 예정

## 순환참조를 다루는 방법
작성 예정

## 구현 방법
### 2-1. Weighted reference counting
### 2-2. Indirect reference counting

## 장점
1. 객체가 접근 불가능해지는 즉시 메모리가 해제되므로, 프로그래머가 객체의 해제 시점을 어느정도 예측 가능하다.
2. 객체가 사용된 직후에 메모리를 해제하므로, 메모리 해제 시점에 해당 객체는 캐시에 저장되어 있을 확률이 높기 때문에 메모리 해제가 빠르게 이루어진다.

## 단점
1. Cycles: 두개 이상의 객체가 서로 참조를 하고있다면, 사이클이 생겨서 참조 횟수가 영원히 0이 될 수 없다. 이를 순환 참조라고 하며, 메모리 누수의 원인이 된다. CPython은 이 문제를 해결하기 위해 순환 참조를 감지하는 알고리즘을 사용한다. 
2. Space overhead (reference count): 레퍼런스 카운팅 방식은 각 객체마다 카운트를 저장할 공간이 필요하다. 이것은 객체의 메모리 영역에서 가까운 곳에 저장될 것인데, 객체마다 32에서 64비트의 공간이 추가적으로 필요로 하게 된다. 일부 시스템에서는 태그가 지정된 포인터를 사용하여 개체 메모리의 사용되지 않는 영역에 참조 횟수를 저장하여 이러한 오버헤드를 완화할 수 있다.
3. Speed overhead (increment/decrement): 참조의 할당이 그 범위를 넘어서면 하나 이상의 reference counter를 수정해야 할 수도 있다. 그러나 참조가 외부 범위 변수에서 내부 범위 변수로 복사되어 내부 변수의 수명이 외부 변수의 수명으로 제한되는 일반적인 경우에는 참조 증가가 제거될 수 있다. 외부변수는 참조를 '소유'한다. C++에서는 const reference를 사용하여 이러한 내용을 쉽게 구현할 수 있다. C++에서 레퍼런스 카운팅은 일반적으로 생성자, 소멸자 및 할당 연산자가 참조를 관리하는 '스마트포인터' 를 사용하여 구현된다.
4. Requires atomicity: 멀티스레드 환경에서, increment/decrement 연산은 atomic operation이어야 한다. 
5. Not real-time: 레퍼런스 카운팅의 간단한 구현체는 real-time으로 사용하기에는 적합하지 않다.

# 3. Escape analysis
이스케이프 분석은 컴파일 타임에 heap allocation을 stack allocation으로 변환하는 방법이다. 이렇게 함으로써 GC의 수집 양을 줄일 수 있다.

# references
- [Garbage collection (computer science)](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))
- [쓰레기 수집 (컴퓨터 과학)](https://ko.wikipedia.org/wiki/%EC%93%B0%EB%A0%88%EA%B8%B0_%EC%88%98%EC%A7%91_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))
