---
title:  "가비지컬렉션(Garbage Collection)의 종류와 특징"
subtitle: "동적 언어의 메모리 관리 기법"
tags:
    - computer science
date:   2021-12-26
---

# Garbage란?

> Garbage는 컴퓨터의 메모리에 있지만 앞으로 사용되지 않을 데이터나 객체 또는 메모리 영역을 가리킵니다.
> 모든 컴퓨터 시스템은 제한적인 메모리를 가지고 있고, 대부분의 소프트웨어 garbage를 만들어내기 때문에, 이를 메모리에서 해제하여 다시 사용할 수 있게 해야 합니다.

아래의 Item과 Stack 두 클래스를 활용한 코드가 다음과 같을때, 두번째로 push해준 Item은 pop 이후 어디에서도 접근할 수 없는 객체, 즉 **garbage**가 되었습니다.

<details>
<summary><span style="cursor: pointer; border-width: 0.1em; border-style: solid; border-color: black">Item, Stack 클래스 (열어서 보기)</span></summary>

```typescript
class Item {
   public value: number;

   constructor(value: number) {
      this.value = value;
   }
}
```

```typescript
class Stack {
   private items: Item[];
   private size: number;

   constructor() {
      this.size = 0;
      this.items = [];
   }

   public push(item: Item): void {
      this.items.push(item);
      this.size++;
   }

   public pop(): Item {
      this.size--;
      this.items.pop();
   }
}
```
</details>

```typescript
const stack: Stack = new Stack();
stack.push(new Item(1)); // [Item(1)]
stack.push(new Item(2)); // [Item(1), Item(2)]
stack.pop(); // [Item(1)] Item(2)
```

---

# Garbage Collection이란?

>가비지컬렉션(garbage collection, GC)은 자동으로 메모리를 관리해주는 기법입니다.
프로세스에 의해 할당되었지만, 더이상 참조되지 않는 메모리를 garbage라고 하고, 이러한 garbage를 수거하는 작업은 garbage collector가 진행합니다.

java, python, javascript 등의 언어들은 이러한 GC를 염두에 두고 설계되어, 언어 자체에 해당 기능이 포함되어 있습니다.<br>
C, C++ 등의 수동 메모리 관리를 가정하고 설계된 언어의 경우에도 GC를 지원하는 구현도 존재합니다.

---

# 1. Tracing
Tracing은 가장 대표적인 GC 기법입니다.

## 기본 알고리즘
Tracing 알고리즘의 기본 개념은 **root object로부터 접근 가능한 객체인지** 추적한다는 것입니다. 
이러한 reference chaining 과정에서 탐지되지 않은 객체는 접근 불가능한 것이고, 따라서 GC의 대상이 됩니다.

### 1-1. mark-and-sweep

이름 그대로 mark(표시)하고 sweep(청소)하는 알고리즘입니다.
mark-and-sweep 알고리즘에서, 각각의 객체는 1비트의 flag를 하나씩 갖습니다.

mark 단계에서는 root set 전체를 순회하며 flag를 '사용중' 상태로 설정합니다.
sweep 단계에서는 메모리가 모두 스캔되었기에 '사용중' 상태가 아닌 메모리를 모두 free로 바꿀 수 있습니다.
이 기법은 몇가지 단점이 있는데, 그중 하나는 GC를 진행하는 동안 전체 시스템을 freeze 시켜야 한다는 점입니다. 

<img src="https://user-images.githubusercontent.com/34048253/147408809-ae9a3134-8b5b-4305-a30f-99fce81509bd.gif" width="60%">

### 1-2. Tri-color marking 

mark-and-sweep의 퍼포먼스 문제(freeze)로 인해, 현대의 tracing GC는 tri-color marking 의 추상적 모델을 기반으로 다양하게 발전하고 있습니다.
tri-color marking은 아래와 같이 동작합니다.

1. 각각의 객체를 흰색, 회색, 검은색으로 분류합니다.
    1. 흰색은 더이상 접근 불가능한 객체를 가리킵니다.
    2. 회색은 접근 가능한 객체이지만, 이 객체에서 가리키는 객체들은 아직 검사되지 않았음을 의미합니다.
    3. 검은색은 이 영역에서 가리키는 객체들이 흰색 객체를 가리키지 않음을 의미합니다.
    4. 알고리즘이 시작할 때는 변수가 가리키는 객체들이 회색으로 표시되며, 그 외의 모든 객체는 흰색으로 표시됩니다. 
2. 회색으로 표시된 객체 가운데 하나를 선택하여 검은색으로 표시하고, 이 객체가 가리키는 모든 객체를 회색으로 표시합니다. 
3. 회색 객체가 하나도 남지 않을 때까지 위 과정을 반복합니다.
4. 남은 흰색 객체는 접근 불가능한 객체이므로, 모두 해제합니다.

이 알고리즘은 mark-and-sweep 알고리즘과는 달리, 상당한 시간동안 시스템의 중단 없이 즉시(on-the-fly) 수행이 가능합니다.

<img src="https://user-images.githubusercontent.com/34048253/147408817-0bff71e7-4528-431d-a127-f04b95326181.gif" width="60%">


## Tracing의 다양한 구현 전략
### Moving vs. non-moving
mark에서 접근 불가능한 객체가 발견되면, garbage collector가 해당 객체를 release하고, 나머지 객체들은 그대로 두거나 새로운 메모리 영역에 copy합니다. 
copy를 하는 경우, 필요에 따라 객체들의 참조를 갱신합니다.

>copy하지 않고 그대로 두는 경우를 `non-moving`이라 하고, copy하는 경우를 `moving`이라 부릅니다. (또는 `non-compacting`과 `compacting` 이라고도 불립니다.)

moving 알고리즘은 non-moving 알고리즘과 비교하여 비교적 비효율적으로 보일 수 있지만, moving 알고리즘은 몇가지 성능적인 이점이 있습니다.
- 제거된 객체의 공간에 대한 추가적인 작업이 필요없습니다. moving 작업 이후 메모리 영역은 여유공간이 되지만, non-moving GC의 경우 release된 객체를 돌아다니며 사용가능하다는 표시를 해야합니다.
- 새로운 객체가 굉장히 빠르게 할당됩니다. moving GC의 경우, 인접영역 대부분이 사용 가능하기 때문에, 새로운 객체를 아주 심플하게 생성할 수 있습니다. non-moving의 경우, 새로운 객체가 인접하게 생성되기 힘들어 매우 무거운 fragmented heap 상태가 되어 관리하기가 더 힘들게 됩니다.  
- 적절한 순회방법이 사용된다면, 객체가 참조하는 객체에 굉장히 인접하게 이동할 수 있습니다. 이로 인해 같은 캐시라인이나 가상메모리 페이지에 위치하게 될 가능성이 높아지게 됩니다. 이로 인해 참조를 통해 객체에 접근할때 굉장히 빠르게 접근할 수 있습니다.

moving GC의 단 하나의 단점은 참조를 통한 접근이 gc 환경에 의해 관리되고, 포인터를 허용하지 않는 점입니다. GC가 객체들을 이동시키게 되면, 포인터가 가리키는 객체는 유효하지 않기 때문입니다.
코드의 상호 운용성을 위해 GC는 객체들을 가비지 수집 영역 외부 위치에 복사해야 합니다. 다른 접근방식은 메모리 내의 객체는 고정하여 GC가 해당 객체를 moving시키지 못하도록 하고, native pointer를 사용하여 직접 메모리에 접근 가능하도록 하는 것이고, 이를 통해 포인터를 허용할 수 있습니다. (자세한 내용은 [Copying and Pinning](http://msdn2.microsoft.com/en-us/library/23acw07k.aspx)을 참고하세요.)

### Copying vs. mark-and-sweep vs. mark-and-don't-sweep
>moving과 non-moving과 같은 차이 뿐만 아니라, marking을 하는 과정에서 white, gray, black객체를 어떻게 다루는지에 대한 차이도 존재합니다.

#### 1. copying (semi-space collector)
**semi-space collector**는 가장 간단한 접근방법입니다. 
여기에서 메모리는 동일한 크기의 **from space**와 **to space**로 나뉘어있습니다.
처음에 모든 객체는 **to space**에 할당되어있고, 이 공간이 꽉차거나, collection cycle이 발동되면, **to space**는 **from space**가 되고, 반대의 경우에도 마찬가지입니다.
root로부터 접근 가능한 객체들은 **from space**에서 **to space**로 복사됩니다. 이후 새로 생성되는 객체는 cycle이 발동되기 전까지 **to space**에 할당됩니다.<br>
이 방법은 심플하지만, 객체를 할당하는 공간이 **from**과 **to**중 한곳만 사용되기 때문에 다른 알고리즘에 비해 메모리 사용량이 두배나 높습니다. 
그리고 이러한 테크닉은 **stop-and-copy**라고도 불립니다.

#### 2. mark and sweep
**mark and sweep** GC는 각 객체별로 white, black 기록을 위해 한개에서 두개의 비트값을 가지게 됩니다. 
gray를 위한 값은 별도로 분리된 리스트를 통해 관리하거나, 객체에 비트를 추가하여 관리할 수 있습니다.
mark and sweep 전략은 moving이나 non-moving 전략중 하나를 선택할 수 있다는 이점이 있습니다.
하지만 객체마다 추가되는 비트로 인해 메모리 비용이 증가하는 단점이 있습니다.

#### 3. mark and don't sweep
**mark and don't sweep** GC는 mark-and-sweep처럼 각 객체에 white, black 정보를 저장하기 위한 비트가 추가됩니다. 
gray또한 마찬가지로 분리된 리스트 또는 객체의 추가비트를 사용하여 관리합니다.
mark and sweep과의 차이점은 크게 두가지 있습니다. 
1. 첫째로, black, white가 mark and sweep에서와는 **다른 의미**를 가집니다. 
mark and don't sweep에서는 **도달 가능한 객체는 항상 black**입니다. 
객체가 할당될 때 black으로 mark되고 **도달 불가능해질 때에도 black**을 유지합니다.
**white는 사용되지 않거나 할당된것같은 객체**에 표시합니다. 
2. 두번째로, **black/white가 뜻하는 의미가 동작중에 바뀝니다.** 
최초 실행시에 0=white, 1=black의 의미를 가진다고 합시다. 
만약 메모리 할당 작업에서 white(사용 가능한 메모리)를 찾지 못했다면, 이는 모든 객체가 black(사용중)이라는 것을 의미합니다.
그 이후 black/white의 의미가 **반전**(0=black, 1=white)되어, 모든 객체의 상태가 1이므로 white가 됩니다.
이것은 접근 가능한 객체가 black이라는 불변성을 일시적으로 꺠뜨리지만, 전체 마킹 단계가 즉시 이어지며 다시 black(0)으로 표시됩니다.
해당 작업이 완료되면 모든 접근 불가능한 메모리는 흰색이 됩니다. 
이로써 sweep 단계가 필요하지 않게 됩니다.

mark and don't sweep 전략은 allocator와 collector의 협업을 필요로 하지만, 할당된 포인터당 1비트만 필요하므로 **공간 효율성**이 매우 높습니다. 
그러나 대부분의 경우 메모리의 많은 부분이 black으로 잘못 표시되어 메모리 사용량이 적은 시간에 시스템에 리소스를 다시 제공하기 어렵기 때문에 이러한 장점이 깎입니다.

결국 **mark and don't sweep** 전략은 **mark and sweep**과 **stop and copy** 전략의 장점을 흡수하고 단점을 보완하여 만들어졌다고 볼 수 있습니다.

### Generational GC (ephemeral GC)
>접근 불가능한 객체들의 **대부분이 생성된지 얼마 안된 것**들이라는 사실이 많은 프로그램을 통해 통계적으로 관찰되었습니다. (Generational hypothesis 라고 알려져 있습니다.)

<img src="https://user-images.githubusercontent.com/34048253/151658996-e4cd9344-8a27-4725-a992-0bb60c50bf23.gif" width="80%" />

Generational GC는 객체들의 세대(generation)를 분리하고, 대부분의 GC 사이클에서 특정 generation에 대해서만 작업을 수행합니다.
또한 런타임 시스템은 참조 생성 및 덮어쓰기를 관찰하여, 객체의 generation이 변경되는 시점에 대한 정보를 유지합니다.
해당 정보를 통해 GC가 동작시에 전체 트리를 순회하지 않고도 어떤 객체들이 GC의 대상인지(특정 generation에 해당하는지) 알 수 있습니다.
만약 generational hypothesis가 성립한다면, 대부분의 GC 타겟들을 회수하면서 수집 사이클을 빠르게 할 수 있습니다.

이러한 컨셉을 구현하기 위해, 많은 generational GC는 메모리의 영역을 분리하여 객체들을 **수명**에 따라 나눠서 관리합니다.
한 메모리 영역이 가득차면, order generation의 참조를 root로 사용하여 해당 영역의 객체들이 추적됩니다.
이때 해당 generation의 대부분의 객체들은 수집되고, 해당 공간은 새로운 객체를 할당하기 위해 비워집니다.
해당 수집 단계에서 많은 객체를 수집하지 못하면, 살아남은 객체들은 기존에 있던 generation에서 다음 영역으로 이동하게 됩니다. 이를 **promotion**이라고 부릅니다. 
그리고 기존 메모리 영역 전체는 새로운 객체들을 위한 공간이 됩니다.
Generational GC는 한 순간에 한 영역만 GC를 진행하면 되기 때문에 굉장히 빠릅니다.

전통적인 generation GC는 두개의 generation을 가졌습니다.<br>
이 GC모델은 new space라고 부르는 **young generation**을 새로운 객체가 생성되는 큰 영역인 `eden`과, 
두개의 작은 공간인 `past survivor space`와, `future survivor space`로 나눕니다.<br>
새 공간의 객체들을 참조할 수 있는 **older generation**에 있는 객체들은 `remembered set`에 보관됩니다.

GC가 일어날때마다, new space에 있는 객체들은 remembered set에 있는 root로부터 추적되고, future survivor space로 복사됩니다.
Future survivor space가 가득차면, 객체들은 오래된 객체들이 머무는 공간으로 promote되고, 이러한 절차를 **tenuring**이라고 부릅니다.
GC가 끝날때, 몇몇 객체들은 future survivor space에 남아있게 되고, eden과 past survivor space는 비어있게 됩니다.
이후 future survivor space와 past survivor space는 서로 교체되고 프로그램을 계속 진행하여 eden에 객체를 할당합니다.
초기 시스템에서는 eden의 크기가 각 survivor space에 비해 5배정도 컸습니다.

<img src="https://user-images.githubusercontent.com/34048253/151661886-34952d3c-3d41-4681-b6e5-9a2bbc28c2a2.PNG" width="70%">

generation GC는 경험적 접근이기 때문에, 몇몇 접근할 수 없는 객체들은 각 GC 사이클에 메모리 반환이 되지 않을 수 있습니다. 
그러므로 주기적으로 full mark and sweep이나 copying GC를 수행해줄 필요가 있습니다.
사실, Java나 .NET Framework과 같은 현대의 프로그래밍 언어의 런타임 시스템은 다양한 전략을 가지는 하이브리드 방식을 사용하고 있습니다.
예를 들어, 대부분의 collection은 몇개의 generation에 대해서만 수행하지만, 가끔 mark-and-sweep이 수행되거나, 드물게 full copying 등이 수행됩니다.
collector 수행 범위를 기준으로 두개의 전략을 나누어 설명할때 **minor cycle(minor gc)**과 **major cycle(major gc)**로 부릅니다.

### Stop-the-world vs. Incremental vs. Concurrent
**Stop-the-world** GC는 collection이 진행되는 동안 프로그램의 실행을 완전히 멈춥니다(**embarrassing pause**).
완전이 멈춤으로써 collector가 동작하는 동안 새로운 객체가 할당되지 않게 되기 때문에 객체들이 갑자기 접근 불가능하게 되는것을 방지합니다.
그러므로 Stop-the-world GC는 상호작용이 없는 프로그램에 더 적합합니다.<br> 
Stop-the-world GC는 구현이 단순하고 Incremental GC방식에 비해 빠릅니다.

**Incremental** GC와 **Concurrent** GC는 메인 프로그램 실행이 중지되는 시간을 줄이는것을 목표로 설계되었습니다.

**Incremental** GC는 별개의 phase에서 GC를 수행하고, 각 phase간에 프로그램 실행을 허용합니다.
하지만 Incremental GC에서 각 phase의 GC 수행시간의 합은 일괄로 GC를 수행한것보다 오래 걸리므로, 
이러한 GC는 총 처리량을 낮출 수 있습니다.

> **Unity - Incremental Garbage Collection**<br>
> Incremental GC 방식은 GC 작업을 여러 개의 슬라이스로 분할합니다. 
> 따라서 GC 작업을 위해 프로그램 실행을 한 번에 오랫동안 중단하지 않고 여러 번에 걸쳐 짧게 중단할 수 있습니다. 
> 이렇게 하면 GC에 소요되는 시간의 총량이 줄어드는 것은 아니지만, 
> 여러 개의 프레임에 걸쳐 워크로드를 분산함으로써 원활한 애니메이션의 맥을 끊는 GC 스파이크 문제를 크게 줄일 수 있습니다.

**Concurrent** GC는 프로그램의 실행 스택을 스캔할때를 제외하고는 프로그램을 멈추지 않습니다.

프로그램과 GC가 서로를 방해하지 않도록 하려면, 설계를 신중하게 해야합니다.
예를들어, 프로그램이 새로운 객체를 할당해야 할때, 런타임 시스템은 GC가 완료될때까지 해당 작업을 연기시키거나, GC에게 이를 알려야합니다.

### Precise vs. conservative and internal pointers
몇몇 collector들은 모든 객체의 포인터(reference)를 정확히 식별(identify)할 수 있습니다.
이러한 collector를 **precise**(또는 exact, accurate) collector라고 부릅니다.

이와 반대되는 collector는 **conservative** 또는 partly conservative collector입니다.
Conservative collector는 메모리의 모든 패턴이 포인터가 될 수 있고, 만약 포인터로 해석된 경우, 그것은 할당된 객체를 가리킬 것이라고 가정합니다.
Conservative collector는 포인터가 아닌것을 포인터로 식별하여(**false positive**), 사용되지 않는 메모리를 해제시키지 않을 수 있습니다.
(하지만 실제로 이러한 문제는 프로그램이 많은 데이터를 처리하지 않는 한 항상 문제가 되는 것은 아닙니다.)

False positive는 일반적으로 64비트 시스템보다 32비트 시스템에서 더 많이 발생합니다.
왜냐하면 64비트의 유효 메모리 주소는 32비트에 비해 비교적 작은 비율을 차지하기 때문입니다.
따라서 임의의 64비트 패턴은 false positive을 발생시킬 가능성이 적습니다.
False negative 또한 발생할 수 있는데, [XOR linked list](https://en.wikipedia.org/wiki/XOR_linked_list)를 사용하는 등의 포인터가 hidden인 경우 등에 해당합니다.
Precise collector가 실용적인지 여부는 해당 프로그래밍 언어의 타입 safety 속성에 달려있습니다.

Conservative GC가 필요한 예로 C 언어를 들 수 있습니다.
C 언어는 type이 지정된(void가 아닌) 포인터를 type이 지정되지 않은(void) 포인터로 [type cast](https://ko.wikipedia.org/wiki/%ED%98%95_%EB%B3%80%ED%99%98)할 수 있고, 그 반대의 경우도 마찬가지입니다.

관련 문제는 **internal pointer** 또는 개체 내의 필드에 대한 포인터와 관련이 있습니다.
만약 언어의 의미 체계가 internal pointer를 허용하는 경우, 동일한 객체에 대해 참조 가능한 다양한 주소값이 있을 수 있고, 이는 해당 객체가 garbage있지 결정하는 것을 복잡하게 만듭니다.
C++의 경우, 다중상속을 사용할 때 base object에 대한 포인터들이 각각 다른 주소를 가질 수 있습니다.
엄격하게 최적화된 프로그램에서, 해당 객체에 대한 포인터가 레지스터에 덮어씌워졌을 수 있으므로, internal pointer를 스캔해야합니다.

# 2. Reference counting
>레퍼런스 카운팅 방식의 GC에서 각 객체는 참조당하는 횟수를 표시해둡니다.<br>
이때 참조 횟수가 0이라면 Garbage라고 할 수 있습니다.<br>
참조가 생기면 count는 증가하고, 참조가 사라지면 감소합니다.<br>
count가 0이 되었을때, 객체를 해제합니다.

## 장점
1. 객체가 접근 불가능해지는 즉시 메모리가 해제되므로, 프로그래머가 객체의 **해제 시점을 어느정도 예측 가능**합니다.
2. 객체가 사용된 직후에 메모리를 해제하므로, 메모리 해제 시점에 해당 객체는 캐시에 저장되어 있을 확률이 높기 때문에 메모리 **해제가 빠르게** 이루어집니다.

## 단점
1. **Cycles**: 두개 이상의 객체가 서로 참조를 하고있다면, 사이클(순환참조)이 생겨서 참조 횟수가 영원히 0이 될 수 없습니다. 
   이를 순환 참조라고 하며, 메모리 누수의 원인이 됩니다. 
   CPython은 이 문제를 해결하기 위해 순환 참조를 감지하는 알고리즘을 사용합니다.
2. **Space overhead (reference count)**: 레퍼런스 카운팅 방식은 각 객체마다 카운트를 저장할 공간이 필요합니다. 
   이것은 객체의 메모리 영역에서 가까운 곳에 저장될 것인데, 객체마다 32에서 64비트의 공간이 추가적으로 필요로 하게 됩니다. 
   일부 시스템에서는 태그가 지정된 포인터를 사용하여 개체 메모리의 사용되지 않는 영역에 참조 횟수를 저장하여 이러한 오버헤드를 완화할 수 있습니다.
3. **Speed overhead (increment/decrement)**: 참조의 할당이 그 범위를 넘어서면 하나 이상의 reference counter를 수정해야 할 수도 있습니다. 
   그러나 참조가 외부 범위 변수에서 내부 범위 변수로 복사되어 내부 변수의 수명이 외부 변수의 수명으로 제한되는 일반적인 경우에는 참조 증가가 제거될 수 있습니다. 
   외부변수는 참조를 '소유'합니다. 
   C++에서는 const reference를 사용하여 이러한 내용을 쉽게 구현할 수 있습니다. 
   C++에서 레퍼런스 카운팅은 일반적으로 생성자, 소멸자 및 할당 연산자가 참조를 관리하는 '스마트포인터' 를 사용하여 구현됩니다.
4. **Requires atomicity**: 멀티스레드 환경에서, 이러한 수정(increment/decrement)은 [compare-and-swap](https://en.wikipedia.org/wiki/Compare-and-swap)과 같은 [atomic operation](https://en.wikipedia.org/wiki/Linearizability)이어야 합니다. 
   멀티 프로세서 환경에서의 atomic operation은 비용이 많이 듭니다.
5. **Not real-time**: 일반적으로 reference counting은 구현할 때 real-time 속성의 동작을 지원하지 않습니다. 
   이는 재귀적인 메모리 해제가 필요한 object가 다른 object 또한 가리키고 있고 (=any pointer assignment), 이 object를 수행중인 스레드는 다른 task를 수행하고 있어 메모리 해제 수행이 불가능한 경우 실시간 처리가 보장되지 않기 때문입니다. 
   이 이슈는 추가적인 연산 오버헤드를 감수하면서 참조되지 않은 object의 메모리 해제를 다른 스레드에 위임하여 회피할 수 있습니다.

## 비효율적인 reference count update를 다루는 방법
>참조가 생성되고 제거될때마다 reference count를 증가,감소 시킨다면 퍼포먼스가 크게 하락할 것입니다.
연산 시간 뿐만 아니라 캐시 퍼포먼스에도 영향을 주고 [pipeline bubble](https://en.wikipedia.org/wiki/Pipeline_stall)이 발생시키는 원인이 됩니다.

1. **묶어서 한꺼번에 업데이트 (nearby reference updates into one)**<br>
   이걸 해결하기 위해 컴파일러가 해줄 수 있는 가장 간단한 방법은, 주변에 reference count가 함께 업데이트 되는 것들을 하나로 묶는 것입니다.
   이것은 reference가 생성된 후 빠른 시일 내에 사라지는 코드에 특히 효율적입니다.
   Deutsch-Bobrow 방법은 대부분의 reference count 업데이트가 지역 변수에 저장된 참조에 의해 생성된다는 사실을 이용합니다.
   지역변수의 참조는 stack 영역의 메모리가 해제되면서 사라지기 때문에 무시할 수 있고, **자료구조(heap 영역에 할당된)**의 참조만 계산하게 됩니다.
   물론 참조 카운트가 0인 개체를 삭제하기 전에 시스템은 스택과 레지스터를 스캔하여 이에 대한 다른 참조가 존재하는지 확인해야합니다.
3. **참조 횟수 증가의 유예 (deferred increments)**<br>
   Henry Baker가 고안한 deferred increments 방법의 경우 지역변수의 참조는 즉시 count를 증가시키지 않고, 필요할때까지 지연시킵니다.
   reference가 빠르게 해제된다면, 카운터를 업데이트 할 필요가 없습니다.
   이러한 방법은 짧게 생겼다 사라지는 reference를 업데이트 하지 않음으로써, 많은 수의 업데이트를 하지 않아도 되게 합니다.
   하지만, 만약 자료구조의 참조가 발생하는 경우, 카운트의 증가를 미루지 않고 즉시 수행합니다.
   또한 count가 0으로 떨어지기 전에 지연된 count 업데이트를 미리 수행하는것도 중요합니다.
4. **업데이트 합치기 (update coalescing method)**<br>
   카운터의 오버헤드를 드라마틱하게 줄이는 방법은 Levanoni와 Petrank에 의해 2001년에 고안되었습니다.
   이 방법은 수많은 쓸모없는 reference count 업데이트를 합치는 것입니다.
   여러번 업데이트 되는 포인터를 상상해봅시다.
   먼저 O1를 가리킨 다음 O2를 가리키며 마지막엔 On을 가리킨다고 할때, reference counting 알고리즘은 rc(O1)--, rc(O2)++, rc(O2)--, rc(O3)++, rc(O3)--, ..., rc(On)++ 와 같은 실행을 보일 것입니다.
   이때 대부분의 업데이트는 불필요하다. rc(O1)--와 rc(On)++면 충분하고, 나머지 업데이트는 불필요합니다.
5. **숨은 레퍼런스 카운팅 (ulterior reference counting method)**<br>
   Blackburn과 McKinley가 2003년에 고안한 이 방법은 deferred reference counting과 copying nursery를 합친 것으로, 포인터 돌연변이의 대부분이 young object에서 발생한다는 것을 관찰했습니다.
   이 알고리즘은 reference counting 중에서도 낮은 pause time을 가지면서도, 가장 빠른 generational copying collector와 필적하는 처리량을 달성했습니다.

## 순환참조를 다루는 방법
순환참조를 다루는 가장 쉬운 방법은 시스템에서 순환참조가 발생하지 않도록 하는 것입니다.
Weak reference(count하지 않는)를 사용하면 cycle을 유지하도록 할 수도 있습니다. Cocoa framework에서는 parent->child 관계에서는 strong reference를 사용하고, child->parent 관계에서는 weak reference를 사용하도록 권장합니다.

순환참조를 참거나(tolerate), 바로 잡도록 하는 방법도 있습니다.
개발자들은 더이상 필요로 하지 않는 레퍼런스들을 해체(tear down)시키도록 코드를 디자인하기도 하지만, 이는 해당 데이터 구조의 수명(lifetime을) 수동으로 추적해야 하는 비용이 듭니다.
이 기술은 Owner 객체를 생성하여 자동화할 수 있습니다. 예를 들어 Graph 객체의 소멸자는 Graph 노드의 엣지를 그래프의 순환구조를 없앨 수도 있습니다.

수명이 짧거나 순환성 garbage가 많지 않은 시스템에서는 Cycle이 무시되기도 합니다.
특히 효율성을 희생하면서 순환구조를 피하는 방법론을 사용하여 개발된 경우에 더욱 그렇습니다.

컴퓨터 과학자들은 자료구조의 설계를 변경할 필요도 없이 순환 참조를 자동으로 탐지하여 수집하는 방법을 발견했는데,
가장 간단한 해결책은 주기적으로 reclaim cycle에 대해 tracing garbage collector를 사용하는 것입니다. 
cycle은 일반적으로 비교적 적은 양의 공간을 사용하므로, 일반적인 tracing garbage collector를 사용하는 것 보다 훨씬 덜 자주 실행할 수 있습니다.

Bacon은 레퍼런스 카운팅의 cycle 수집 알고리즘은 tracing collector와 유사하다고 묘사했습니다.
이는 레퍼런스 카운트가 0이 아닌 값으로 감소할때만 사이클을 분리할 수 있다는 관찰을 기반으로 합니다.
이에 해당하는 모든 객체는 루트 list에 배치되고, 프로그램은 주기적으로 루트에서 도달할 수 있는 객체를 통해 Cycle을 검색합니다.
순환참조에 대한 모든 레퍼런스 카운트를 감소시키면, 수집될 수 있는 사이클이 발견됩니다.
이 알고리즘의 향상된 버전은 다른 작업들과 동시에 실행할 수 있으며, update coalescing 방법을 사용하여 효율성을 향상시킬 수 있습니다.

## 구현 방법
레퍼런스 카운트를 증가시키는 방법은 다양합니다. 이중 효율이 좋은 레퍼런스 카운팅 방법 두가지는 동작 방식이 근본적으로 다릅니다.
아래의 두가지 방법의 장단점을 살펴봅시다.

### 2-1. Weighted reference counting
Weighted reference counting은 1987년 Bevan과 Watson & Watson에 의해 고안된 방식입니다.
Weighted reference counting에서 각 레퍼런스는 weight를 가지고있고,
각 객체는 레퍼런스 카운트를 가지고 있는 대신 레퍼런스를 추적하기 때문에 추적하고 있는 모든 레퍼런스의 weight인 total weight을 갖게 됩니다.
새로 생긴 객체를 참조할때는 2^16과 같은 큰 weight를 가지게 됩니다. 
reference가 복사될때, weight의 절반이 새로운 레퍼런스로 이동하게 됩니다. 
그리고 나머지 절반의 weight은 기존 레퍼런스에 남아있게 됩니다.
total weight은 변하지 않기 때문에, 해당 객체의 reference count는 업데이트할 필요가 없습니다.

Reference를 제거할때는 total weight에 제거할 레퍼런스가 가지는 weight만큼을 빼줍니다. 
total weight가 0이 되면, 모든 레퍼런스가 삭제된 것입니다.
만약 weight가 1인 객체를 복사하려 하면, 해당 레퍼런스는 total weight을 증가시켜야 하고, 증가된 만큼을 새로운 레퍼런스의 weight로 합니다.

위의 방법을 다른식으로도 구현할 수 있는데, 간접 참조 객체를 생성하는 것입니다. 
이때 간첩 참조로 생성된 최초의 레퍼런스는 기존 weight의 split 대신 large weight(ex. 2^16)을 가지게 됩니다.

레퍼런스가 복사될때 레퍼런스 카운트에 접근할 필요가 없는 이러한 속성은 레퍼런스 카운트에 접근하는 비용이 큰 객체의 경우에 특히 유리합니다.
레퍼런스 카운트 비용이 큰 예시는 disk나 network가 있습니다.

이러한 방식은 많은 스레드가 레퍼런스 카운트를 증가시키기 위해 참조를 locking 하는 것을 피할 수 있게 해줍니다. 
그렇기 때문에, weighted reference counting 방식은 병렬처리, 멀티프로세스, 데이터베이스, 분산처리 애플리케이션 등에 효율적입니다.

Simple reference counting 방식의 주요한 문제는 참조를 해제하기 위해 레퍼런스 카운트에 접근해야 한다는 점이고, 많은 레퍼런스가 해제되는 상황에서 병목 현상이 발생하게 됩니다.
Weighted reference counting 방식에서는 죽어가는 레퍼런스의 weight를 액티브 레퍼런스로 옮기는 식으로 이러한 문제를 회피합니다.

### 2-2. Indirect reference counting
Indirect reference counting(IRC) 는 레퍼런스 카운팅을 사용하는 분산된 비순환 구조의 가비지컬렉션의 재정렬 문제를 해결하기 위해 고안되었습니다.
실제로 레퍼런스 카운트는 프로세스 전체에 퍼져있습니다.
트리 구조는 현재 개체에 대한 레퍼런스를 보유하고 있는 모든 leaf node와 레퍼런스 카운트의 일부를 보유하고 있는 모든 내부 노드와 함께 이러한 카운트를 연결합니다.
count의 감소는 트리의 분기를 따라 전달되지만, 증가는 로컬에만 해당됩니다.
따라서 기본 커뮤니케이션 레이어의 재정렬로 인해 레퍼런스 카운트의 증가/감소에 따른 race condition이 발생하지 않습니다.

IRC를 고안한 Piquer는 두개의 참조를 사용하여 remote reference를 구현했는데,
두 참조중 하나는(논문에서 primary라고 부른다.) 원하는 리모트 객체를 가리키고,
다른 하나는(여기서는 source pointer라고 부른다.) 얻어진 참조의 원본(source)을 가리킵니다.
remote reference가 복사될때 primary reference는 원본의 복사본의 참조와 동일하지만,
source pointer는 복사본이 생성되는 노드를 나타내고 해당 노드에 대한 로컬 레퍼런스 카운트가 증가합니다.
remote reference가 버려질때, dec 메시지가 source node로 전송됩니다.
주어진 객체의 remote reference에 대한 source pointer는 해당 객체를 root로 한 diffusion tree를 형성합니다.

아래의 참조관계를 보면, B와 C는 A로부터 직접참조가 되지만, D는 A로부터 간접참조가 되어있습니다.

<img src="https://user-images.githubusercontent.com/34048253/147879591-8965f11e-0281-417c-9299-f625d69ee12c.png" width="35%" />

위 참조관계를 Standard Reference Counting과 Indirect Reference Counting 방식으로 풀어봅시다.

<img src="https://user-images.githubusercontent.com/34048253/147879670-0e68e772-5382-4b48-8ca1-086b8ada0454.png" width="70%">

- 왼쪽 이미지는 일반적인 레퍼런스 카운팅입니다. (Standard Naive reference counting)
- 우측 이미지는 Indirect reference counting으로, 모든 remote reference는 연관된 복사 수가 있고, primary 포인터와 다른 source pointer가 있을 수 있습니다.

# 3. Escape analysis
이스케이프 분석은 컴파일 시점에 [heap allocation](https://en.wikipedia.org/wiki/Heap_allocation)을 [stack allocation](https://en.wikipedia.org/wiki/Stack-based_memory_allocation)으로 변환하는 방법입니다. 
이렇게 함으로써 GC의 수집 양을 줄일 수 있습니다.

예를 들어, Swift에서는 ARC(Automatic Reference Counting)이라는 메모리 관리 기법을 통해 컴파일 시점에 컴파일러가 개발자를 대신하여 메모리 관리 코드를 적절한 위치에 자동으로 삽입합니다.
즉, Swift 코드에서 개발자가 release 코드를 직접 작성하지 않아도 실제로 바이너리 코드에는 메모리 해제 코드가 들어가도록 한다는 것입니다.

# 언어별 GC 문서
위의 GC 종류에 대해 쭉 따라오셨다면, 다음 언어별 GC 문서에 대해 읽을때 전보다 이해가 더 잘 될 것입니다. 
- [Java GC Basic](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
- [Java G1GC](https://www.oracle.com/java/technologies/javase/hotspot-garbage-collection.html)
- [Python](http://www.arctrix.com/nas/python/gc/)
- [Javascript](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management)
- [C++, NodeJS](https://chromium.googlesource.com/v8/v8.git/+/HEAD/include/cppgc/#oilpan_c_garbage-collection)
- [Golang](https://github.com/golang/go/blob/master/src/runtime/mgc.go)

# references
- [Garbage collection (computer science) - wikipedia](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))
- [쓰레기 수집 (컴퓨터 과학) - 위키피디아](https://ko.wikipedia.org/wiki/%EC%93%B0%EB%A0%88%EA%B8%B0_%EC%88%98%EC%A7%91_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))
- [Diffusion Tree Restructuring for Indirect Reference Counting](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.30.2813&rep=rep1&type=pdf)
- [Automatic Reference Counting - swift](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)
- [Java Garbage Collection - Naver D2](https://d2.naver.com/helloworld/1329)
