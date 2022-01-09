---
title:  "가비지컬렉션(Garbage Collection)의 종류와 특징"
subtitle: "동적 언어의 메모리 관리 기법"
tags:
    - computer science
date:   2021-12-26
---

> **아직 작성되지 않은 부분이 많이 있습니다.**<br>
> **빠르게 채워넣도록 하겠습니다.**

가비지컬렉션 (garbage collection, GC)은 자동으로 메모리를 관리해주는 기법이다.
프로세스에 의해 할당되었지만, 더이상 참조되지 않는 메모리를 garbage라고 하고, 이러한 garbage를 수거하는 작업은 garbage collector가 진행한다.

java, python 등의 언어들은 이러한 GC를 염두에 두고 설계되어, 언어 자체에 해당 기능이 포함되어 있다.<br>
C, C++ 등의 수동 메모리 관리를 가정하고 설계된 언어의 경우에도 GC를 지원하는 구현도 존재한다.

# 1. Tracing
tracing은 가장 대표적인 GC 기법이다.
tracing 알고리즘의 기본 개념은 root object로부터 접근 가능한 객체인지 추적한다는 것이다. 이러한 reference chaining 과정에서 탐지되지 않은 객체는 접근 불가능한 것이고, 따라서 GC의 대상이 된다.

## 기본 알고리즘 1-1. mark-and-sweep

이름 그대로 mark하고 sweep하는 알고리즘이다.
mark-and-sweep 알고리즘에서, 각각의 객체는 1비트의 flag를 하나씩 갖는다.

mark 단계에서는 root set 전체를 순회하며 flag를 '사용중' 상태로 설정한다.
sweep 단계에서는 메모리가 모두 스캔되었기에 '사용중' 상태가 아닌 메모리를 모두 free로 바꿀 수 있다.
이 기법은 몇가지 단점이 있는데, 그중 하나는 GC를 진행하는 동안 전체 시스템을 freeze 시켜야 한다는 점이다. 

![Animation_of_the_Naive_Mark_and_Sweep_Garbage_Collector_Algorithm](https://user-images.githubusercontent.com/34048253/147408809-ae9a3134-8b5b-4305-a30f-99fce81509bd.gif)

## 기본 알고리즘 1-2. Tri-color marking 

mark-and-sweep의 퍼포먼스 문제로 인해, 현대의 tracing GC는 tri-color marking 의 추상적 모델을 기반으로 다양하게 발전하고 있다.
tri-color marking은 아래와 같이 동작한다.

1. 각각의 객체를 흰색, 회색, 검은색으로 분류한다.
    1. 흰색은 더이상 접근 불가능한 객체를 가리킨다.
    2. 회색은 접근 가능한 객체이지만, 이 객체에서 가리키는 객체들은 아직 검사되지 않았음을 의미한다.
    3. 검은색은 이 영역에서 가리키는 객체들이 흰색 객체를 가리키지 않음을 의미한다.
    4. 알고리즘이 시작할 때는 변수가 가리키는 객체들이 회색으로 표시되며, 그 외의 모든 객체는 흰색으로 표시된다. 
2. 회색으로 표시된 객체 가운데 하나를 선택하여 검은색으로 표시하고, 이 객체가 가리키는 모든 객체를 회색으로 표시한다. 
3. 회색 객체가 하나도 남지 않을 때까지 위 과정을 반복한다.
4. 남은 흰색 객체는 접근 불가능한 객체이므로, 모두 해제한다.

이 알고리즘은 mark-and-sweep 알고리즘과는 달리, 상당한 시간동안 시스템의 중단 없이 즉시(on-the-fly) 수행이 가능하다.

![Animation_of_tri-color_garbage_collection](https://user-images.githubusercontent.com/34048253/147408817-0bff71e7-4528-431d-a127-f04b95326181.gif)

## 다양한 구현 전략
### Moving vs. non-moving
작성 예정
### Copying vs. mark-and-sweep vs. mark-and-don't-sweep
작성 예정
### Generational GC (ephemeral GC)
작성 예정
### Stop-the-world vs. incremental vs. concurrent
작성 예정
### Precise vs. conservative and internal pointers
작성 예정

# 2. Reference counting
레퍼런스 카운팅 방식의 GC에서 각 객체는 참조당하는 횟수를 표시해둔다.
이때 참조 횟수가 0이라면 Garbage라고 할 수 있다.
참고가 생기면 count는 증가하고, 참조가 사라지면 감소한다.
count가 0이 되었을때, 객체를 해제한다.

## 장점
1. 객체가 접근 불가능해지는 즉시 메모리가 해제되므로, 프로그래머가 객체의 해제 시점을 어느정도 예측 가능하다.
2. 객체가 사용된 직후에 메모리를 해제하므로, 메모리 해제 시점에 해당 객체는 캐시에 저장되어 있을 확률이 높기 때문에 메모리 해제가 빠르게 이루어진다.

## 단점
1. **Cycles**: 두개 이상의 객체가 서로 참조를 하고있다면, 사이클(순환참조)이 생겨서 참조 횟수가 영원히 0이 될 수 없다. 이를 순환 참조라고 하며, 메모리 누수의 원인이 된다. CPython은 이 문제를 해결하기 위해 순환 참조를 감지하는 알고리즘을 사용한다.
2. **Space overhead (reference count)**: 레퍼런스 카운팅 방식은 각 객체마다 카운트를 저장할 공간이 필요하다. 이것은 객체의 메모리 영역에서 가까운 곳에 저장될 것인데, 객체마다 32에서 64비트의 공간이 추가적으로 필요로 하게 된다. 일부 시스템에서는 태그가 지정된 포인터를 사용하여 개체 메모리의 사용되지 않는 영역에 참조 횟수를 저장하여 이러한 오버헤드를 완화할 수 있다.
3. **Speed overhead (increment/decrement)**: 참조의 할당이 그 범위를 넘어서면 하나 이상의 reference counter를 수정해야 할 수도 있다. 그러나 참조가 외부 범위 변수에서 내부 범위 변수로 복사되어 내부 변수의 수명이 외부 변수의 수명으로 제한되는 일반적인 경우에는 참조 증가가 제거될 수 있다. 외부변수는 참조를 '소유'한다. C++에서는 const reference를 사용하여 이러한 내용을 쉽게 구현할 수 있다. C++에서 레퍼런스 카운팅은 일반적으로 생성자, 소멸자 및 할당 연산자가 참조를 관리하는 '스마트포인터' 를 사용하여 구현된다.
4. **Requires atomicity**: 멀티스레드 환경에서, 이러한 수정(increment/decrement)은 [compare-and-swap](https://en.wikipedia.org/wiki/Compare-and-swap)과 같은 [atomic operation](https://en.wikipedia.org/wiki/Linearizability)이어야 한다. 멀티 프로세서 환경에서의 atomic operation은 비용이 많이 든다.
5. **Not real-time**: 일반적으로 reference counting은 구현할 때 real-time 속성의 동작을 지원하지 않는다. 이는 재귀적인 메모리 해제가 필요한 object가 다른 object 또한 가리키고 있고 (=any pointer assignment), 이 object를 수행중인 스레드는 다른 task를 수행하고 있어 메모리 해제 수행이 불가능한 경우 실시간 처리가 보장되지 않기 때문이다. 이 이슈는 추가적인 연산 오버헤드를 감수하면서 참조되지 않은 object의 메모리 해제를 다른 스레드에 위임하여 회피할 수 있다.

## 비효율적인 reference count update를 다루는 방법
참조가 생성되고 제거될때마다 reference count를 증가,감소 시킨다면 퍼모먼스가 크게 하락할 것이다.
연산 시간 뿐만 아니라 캐시 퍼포먼스에도 영향을 주고 [pipeline bubble](https://en.wikipedia.org/wiki/Pipeline_stall)이 발생시키는 원인이 된다.

1. **묶어서 한꺼번에 업데이트 (nearby reference updates into one)**<br> 
이걸 해결하기 위해 컴파일러가 해줄 수 있는 가장 간단한 방법은, 주변에 reference count가 함께 업데이트 되는 것들을 하나로 묶는 것이다. 
이것은 reference가 생성된 후 빠른 시일 내에 사라지는 코드에 특히 효율적이다.
2. **지역변수의 참조는 무시 (ignore local variables)**<br> 
Deutsch-Bobrow 방법은 대부분의 reference count 업데이트가 지역 변수에 저장된 참조에 의해 생성된다는 사실을 이용한다. 
지역변수의 참조는 stack 영역의 메모리가 해제되면서 사라지기 때문에 무시할 수 있고, **자료구조(heap 영역에 할당된)**의 참조만 계산하게 된다. 
물론 참조 카운트가 0인 개체를 삭제하기 전에 시스템은 스택과 레지스터를 스캔하여 이에 대한 다른 참조가 존재하는지 확인해야한다.
3. **참조 횟수 증가의 유예 (deferred increments)**<br>
Henry Baker가 고안한 deferred increments 방법의 경우 지역변수의 참조는 즉시 count를 증가시키지 않고, 필요할때까지 지연시킨다. 
reference가 빠르게 해제된다면, 카운터를 업데이트 할 필요가 없다. 
이러한 방법은 짧게 생겼다 사라지는 reference를 업데이트 하지 않음으로써, 많은 수의 업데이트를 하지 않아도 되게 한다. 
하지만, 만약 자료구조의 참조가 발생하는 경우, 카운트의 증가를 미루지 않고 즉시 수행한다. 
또한 count가 0으로 떨어지기 전에 지연된 count 업데이트를 미리 수행하는것도 중요하다.
4. **업데이트 합치기 (update coalescing method)**<br>
카운터의 오버헤드를 드라마틱하게 줄이는 방법은 Levanoni와 Petrank에 의해 2001년에 고안되었다. 
이 방법은 수많은 쓸모없는 reference count 업데이트를 합치는 것이다. 
여러번 업데이트 되는 포인터를 상상해보자. 
먼저 O1를 가리킨 다음 O2를 가리키며 마지막엔 On을 가리킨다고 할때, reference counting 알고리즘은 rc(O1)--, rc(O2)++, rc(O2)--, rc(O3)++, rc(O3)--, ..., rc(On)++ 와 같은 실행을 보일 것이다. 
이때 대부분의 업데이트는 불필요하다. rc(O1)--와 rc(On)++면 충분하고, 나머지 업데이트는 불필요하다.
5. **숨은 레퍼런스 카운팅 (ulterior reference counting method)**<br>
Blackburn과 McKinley가 2003년에 고안한 이 방법은 deferred reference counting과 copying nursery를 합친 것으로, 포인터 돌연변이의 대부분이 young object에서 발생한다는 것을 관찰했다.
이 알고리즘은 reference counting 중에서도 낮은 pause time을 가지면서도, 가장 빠른 generational copying collector와 필적하는 처리량을 달성했다.

## 순환참조를 다루는 방법
순환참조를 다루는 가장 쉬운 방법은 시스템에서 순환참조가 발생하지 않도록 하는 것입니다.
Weak reference(count하지 않는)를 사용하면 cycle을 유지하도록 할 수도 있습니다. Cocoa framework에서는 parent->child 관계에서는 strong reference를 사용하고, child->parent 관계에서는 weak reference를 사용하도록 권장합니다.

순환참조를 참거나(tolerate), 바로 잡도록 하는 방법도 있습니다. 
개발자들은 더이상 필요로 하지 않는 레퍼런스들을 해체(tear down)시키도록 코드를 디자인하기도 하지만, 이는 해당 데이터 구조의 수명(lifetime을) 수동으로 추적해야 하는 비용이 듭니다.
이 기술은 Owner 객체를 생성하여 자동화할 수 있습니다. 예를 들어 Graph 객체의 소멸자는 Graph 노드의 엣지를 그래프의 순환구조를 없앨 수도 있습니다.

수명이 짧거나 순환성 garbage가 많지 않은 시스템에서는 Cycle이 무시되기도 합니다. 
특히 효율성을 희생하면서 순환구조를 피하는 방법론을 사용하여 개발된 경우에 더욱 그렇습니다. 

컴퓨터 과학자들은 자료구조의 설계를 변경할 필요도 없이 순환 참조를 자동으로 탐지하여 수집하는 방법을 발견했는데, 
가장 간단한 해결책은 주기적으로 reclaim cycle에 대해 tracing garbage collector를 사용하는 것입니다. cycle은 일반적으로 비교적 적은 양의 공간을 사용하므로, 일반적인 tracing garbage collector를 사용하는 것 보다 훨씬 덜 자주 실행할 수 있습니다.

Bacon은 레퍼런스 카운팅의 cycle 수집 알고리즘은 tracing collector와 유사하다고 묘사했습니다. 
이는 레퍼런스 카운트가 0이 아닌 값으로 감소할때만 사이클을 분리할 수 있다는 관찰을 기반으로 합니다.
이에 해당하는 모든 객체는 루트 list에 배치되고, 프로그램은 주기적으로 루트에서 도달할 수 있는 객체를 통해 Cycle을 검색합니다.
순환참조에 대한 모든 레퍼런스 카운트를 감소시키면, 수집될 수 있는 사이클이 발견됩니다. 
이 알고리즘의 향상된 버전은 다른 작업들과 동시에 실행할 수 있으며, update coalescing 방법을 사용하여 효율성을 향상시킬 수 있습니다.

## 구현 방법
레퍼런스 카운트를 증가시키는 방법은 다양하다, 이중 효율이 좋은 레퍼런스 카운팅 방법 두가지는 동작 방식이 근본적으로 다르다.
아래의 두가지 방법의 장단점을 살펴보자.

### 2-1. Weighted reference counting
Weighted reference counting은 1987년 Bevan과 Watson & Watson에 의해 고안된 방식이다.
Weighted reference counting에서 각 레퍼런스는 weight를 가지고있고, 
각 객체는 레퍼런스 카운트를 가지고 있는 대신 레퍼런스를 추적하기 때문에 추적하고 있는 모든 레퍼런스의 weight인 total weight을 갖게 된다.
새로 생긴 객체를 참조할때는 2^16과 같은 큰 weight를 가지게 된다. reference가 복사될때, weight의 절반이 새로운 레퍼런스로 이동하게 된다. 그리고 나머지 절반의 weight은 기존 레퍼런스에 남아있게 된다.
total weight은 변하지 않기 때문에, 해당 객체의 reference count는 업데이트할 필요가 없다.

Reference를 제거할때는 total weight에 제거할 레퍼런스가 가지는 weight만큼을 빼준다. total weight가 0이 되면, 모든 레퍼런스가 삭제된 것이다.
만약 weight가 1인 객체를 복사하려 하면, 해당 레퍼런스는 total weight을 증가시켜야 하고, 증가된 만큼을 새로운 레퍼런스의 weight로 한다.

위의 방법을 다른식으로도 구현할 수 있는데, 간접 참조 객체를 생성하는 것이다. 이때 간첩 참조로 생성된 최초의 레퍼런스는 기존 weight의 split 대신 large weight(ex. 2^16)을 가지게 된다.

레퍼런스가 복사될때 레퍼런스 카운트에 접근할 필요가 없는 이러한 속성은 레퍼런스 카운트에 접근하는 비용이 큰 객체의 경우에 특히 유리하다.
레퍼런스 카운트 비용이 큰 예시는 disk나 network가 있다.

이러한 방식은 많은 스레드가 레퍼런스 카운트를 증가시키기 위해 참조를 locking 하는 것을 피할 수 있게 해준다. 그렇기 때문에, weighted reference counting 방식은 병렬처리, 멀티프로세스, 데이터베이스, 분산처리 애플리케이션 등에 효율적이다.

Simple reference counting 방식의 주요한 문제는 참조를 해제하기 위해 레퍼런스 카운트에 접근해야 한다는 점이고, 많은 레퍼런스가 해제되는 상황에서 병목 현상이 발생하게 된다.
Weighted reference counting 방식에서는 죽어가는 레퍼런스의 weight를 액티브 레퍼런스로 옮기는 식으로 이러한 문제를 회피한다.

### 2-2. Indirect reference counting
Indirect reference counting(IRC) 는 레퍼런스 카운팅을 사용하는 분산된 비순환 구조의 가비지컬렉션의 재정렬 문제를 해결하기 위해 고안되었다.
실제로 레퍼런스 카운트는 프로세스 전체에 퍼져있다. 
트리 구조는 현재 개체에 대한 레퍼런스를 보유하고 있는 모든 leaf node와 레퍼런스 카운트의 일부를 보유하고 있는 모든 내부 노드와 함께 이러한 카운트를 연결한다.
count의 감소는 트리의 분기를 따라 전달되지만, 증가는 로컬에만 해당된다.
따라서 기본 커뮤니케이션 레이어의 재정렬로 인해 레퍼런스 카운트의 증가/감소에 따른 race condition이 발생하지 않는다.

IRC를 고안한 Piquer는 두개의 참조를 사용하여 remote reference를 구현했는데, 
두 참조중 하나는(논문에서 primary라고 부른다.) 원하는 리모트 객체를 가리키고, 
다른 하나는(여기서는 source pointer라고 부른다.) 얻어진 참조의 원본(source)을 가리킨다.
remote reference가 복사될때 primary reference는 원본의 복사본의 참조와 동일하지만, 
source pointer는 복사본이 생성되는 노드를 나타내고 해당 노드에 대한 로컬 레퍼런스 카운트가 증가한다.
remote reference가 버려질때, dec 메시지가 source node로 전송된다.
주어진 객체의 remote reference에 대한 source pointer는 해당 객체를 root로 한 diffusion tree를 형성한다.

아래의 참조관계를 보면, B와 C는 A로부터 직접참조가 되지만, D는 A로부터 간접참조가 되어있다.

<img src="https://user-images.githubusercontent.com/34048253/147879591-8965f11e-0281-417c-9299-f625d69ee12c.png" width=300 />

위 참조관계를 Standard Reference Counting과 Indirect Reference Counting 방식으로 풀어보자.

<img src="https://user-images.githubusercontent.com/34048253/147879670-0e68e772-5382-4b48-8ca1-086b8ada0454.png" width="600">

- 왼쪽 이미지는 일반적인 레퍼런스 카운팅이다. (Standard Naive reference counting)
- 우측 이미지는 Indirect reference counting으로, 모든 remote reference는 연관된 복사 수가 있고, primary 포인터와 다른 source pointer가 있을 수 있다.

# 3. Escape analysis
이스케이프 분석은 컴파일 타임에 [heap allocation](https://en.wikipedia.org/wiki/Heap_allocation)을 [stack allocation](https://en.wikipedia.org/wiki/Stack-based_memory_allocation)으로 변환하는 방법이다. 이렇게 함으로써 GC의 수집 양을 줄일 수 있다.

# references
- [Garbage collection (computer science)](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))
- [쓰레기 수집 (컴퓨터 과학)](https://ko.wikipedia.org/wiki/%EC%93%B0%EB%A0%88%EA%B8%B0_%EC%88%98%EC%A7%91_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))
- [Diffusion Tree Restructuring for Indirect Reference Counting](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.30.2813&rep=rep1&type=pdf)
