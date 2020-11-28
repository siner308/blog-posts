---
layout: post
title:  "[번역] 그림으로 보는 SOLID 원칙"
subtitle: "객체지향 프로그래밍 5대 원칙"
author: "Siner"
header-img: "img/post_headers/2020-06-18-solid-principles.png"
catalog: true
header-mask:  0.3
tags:
    - Architecture
    - OOP
date:   2020-06-18
multilingual: false
---

>**SOLID 원칙과 관련된 좋은 그림예시가 있어서 이를 번역하면서 예제코드를 추가하였습니다.**

만약 당신이 `객체지향 프로그래밍`과 친숙하다면, 당신은 `SOLID 원칙`에 대해 들어보았을 것 입니다.

이러한 다섯가지 개발 원칙은 소프트웨어 개발과 **유지보수를 쉽게 할수 있도록 하는 가이드라인**이라고 볼 수 있습니다.
이러한 원칙은 [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin)이라고 하는 소프트웨어 엔지니어에 의해 유명해졌습니다.

SOLID 원칙과 관련된 좋은 자료들이 온라인에 많이 존재하지만, 이것을 **그림으로 설명하는 글**을 찾아보기는 힘듭니다. 이 때문에 `시각적 학습`을 선호하는 이들에게는 어려움이 따르고 있습니다.

그렇기 떄문에, 이 포스팅의 목적은 **그림을 통해 각각의 SOLID 원칙을 쉽게 이해하는 것** 입니다.

이미 알고있겠지만, 각각의 원칙은 비슷하게 보이면서도 서로 다른 목표를 바라보고 있습니다. 한 원칙을 만족하는 코드를 작성하더라도, 다른 원칙에 위배될 가능성이 충분히 있습니다.

글에는 이해하기 쉽도록 `클래스`라는 단어를 사용하지만, 이는 **함수**, **메서드**, **모듈**에도 적용이 가능합니다.

이제 시작해봅시다!

### SRP: 단일 책임 원칙 (Single Responsibility Principle)

>클래스는, 오직 하나의 대해서만 책임져야 한다.

![image](/posts/images/solid-principles/srp-1.png)
![image](/posts/images/solid-principles/srp-2.jpg)

만약 클래스가 여러가지 작업을 책임져야 한다면, 이는 버그 발생 가능성을 높입니다.
당신이 많은 기능중 한가지를 변경할때, 당신이 모르는 사이에 다른 기능에 영향을 줄 수 있기 때문입니다.

>SRP의 목적은 행동들을 분리하는 것이고, 이로 인해 당신이 어떤 기능을 수정하더라도, 연관없는 기능에는 영향이 가지 않게 될 것입니다.

#### 예제 코드 (Bad Case)

아래처럼 하나의 클래스가 여러가지 일을 하게 되는 경우 position이 계속해서 변경되고, 좀더 복잡한 비즈니스 로직에서는 문제가 발생할 확률이 대단히 높습니다.
```typescript
class Generalist {
  public position: string;

  public developBackend(): string {
    // 백엔드 개발을 한다
    this.position = 'backend-developer';
    return this.position;
  }

  public developFrontend(): string {
    // 프론트 개발을 한다
    this.position = 'frontend-developer';
    return this.position;
  }

  public design(): string {
    // 디자인을 한다
    this.position = 'design';
    return this.position;
  }

  public marketing(): string {
    // 마케팅을 한다
    this.position = 'marketing';
    return this.position;
  }
}

const worker: Generalist = new Generalist(); // 나는 제너럴리스트다 (라고 쓰고 개잡부라고 읽는다)

worker.developBackend();
worker.developFrontend();
worker.design();
worker.marketing();
```

#### 예제 코드 (Good Case)
아래의 예시에서는 역할별로 클래스를 분리하여 사용하게 됩니다.

```typescript
export class BasicWorker {
  public position;

  public doMyJob(): boolean {
    // 내 할일만 하면 된다
    try {
      return this.position;
    } catch {
      throw Error('I don\'t have any position'); 
    }
  }
}

class BackendDeveloper extends BasicWorker {
  constructor() {
    super();
    this.position = 'backend-developer';
  }
}

class FrontendDeveloper extends BasicWorker {
  constructor() {
    super();
    this.position = 'frontend-developer';
  }
}

class Designer extends BasicWorker {
  constructor() {
    super();
    this.position = 'designer';
  }
}

class Marketer extends BasicWorker {
  constructor() {
    super();
    this.position = 'marketer';
  }
}

const backendDeveloper: BackendDeveloper = new BackendDeveloper(); // 나는 백엔드 개발자다.
backendDeveloper.doMyJob();

const frontendDeveloper: FrontendDeveloper = new FrontendDeveloper(); // 나는 프론트엔드 개발자다.
frontendDeveloper.doMyJob();

const designer: Designer = new Designer(); // 나는 디자이너다.
designer.doMyJob();

const marketer: Marketer = new Marketer(); // 나는 마케터다.
marketer.doMyJob();
```

### OCP: 개방-폐쇄 원칙 (Open-Closed Principle)

> 클래스는 확장에는 개방적이어야 하고, 변경에는 폐쇄적이어야 한다.

![image](/posts/images/solid-principles/ocp-1.png)
![image](/posts/images/solid-principles/ocp-2.jpg)

클래스의 현재 코드를 변경하는것은 해당 클래스를 사용하고 있는 모든 시스템에 영향을 주게 됩니다. 

만약 클래스에 더 많은 기능을 부여하고 싶다면, 가장 이상적인 접근방법은 기존 기능을 변경하는것이 아닌 새로운 함수를 추가하는 것 입니다.

>OCP의 목적은 클래스의 존재하는 기능의 변경 없이 해당 클래스의 기능을 확장시키는 것 입니다.
>이로인해 사용중인 클래스의 변경으로 인한 버그 발생을 피할 수 있습니다.

#### 예제 코드 (Bad Case)

나쁜 개발자는 먹고 자고 개발하고 쉬는것을 한번에 이어서 진행합니다.
```typescript
class BadDeveloper2019 {
  public eatSleepDevelopRest(): boolean {
    console.log('eating'); // 먹고
    console.log('sleeping'); // 자고
    console.log('developing'); // 개발하고
    console.log('resting'); // 쉬고
    return true
  }
}

const developer = new BadDeveloper2019();
developer.eatSleepDevelopRest();

/**
* 중간에 다른일을 추가해야한다면?
*/

class BadDeveloper2020 {
  public eatSleepDevelopSomeWorksRest(): boolean {
    console.log('eating'); // 먹고
    console.log('sleeping'); // 자고
    console.log('developing'); // 개발하고
    console.log('some job 1'); // 다른일 1
    console.log('some job 2'); // 다른일 2
    console.log('resting'); // 쉬고
    return true
  }
}

const developer = new BadDeveloper2020();
developer.eatSleepDevelopSomeWorksRest(); // 기존의 함수를 변경하여 새로운 함수가 탄생해버렸다...
```

#### 예제 코드 (Good Case)
좋은 개발자는 각 행동들을 나누어 관리하며, 원하는 시기에 수행할 수 있고, 중간에 다른 업무를 넣을 수도 있습니다.
```typescript
class GoodDeveloper2019 {
  public eat(): boolean {
    console.log('eating'); // 먹기
    return true;
  }

  public sleep(): boolean {
    console.log('sleeping'); // 자기
    return true;
  }

  public develop(): boolean {
    console.log('developing'); // 개발하기
    return true;
  }

  public rest(): boolean {
    console.log('resting'); // 쉬기
    return true;
  }
}

const developer = new GoodDeveloper2019();
developer.eat();
developer.sleep();
developer.develop();
developer.rest();

/**
* 중간에 다른일을 추가해야한다면?
*/

class GoodDeveloper2020 {
  public eat(): boolean {
    console.log('eating'); // 먹기
    return true;
  }

  public sleep(): boolean {
    console.log('sleeping'); // 자기
    return true;
  }

  public develop(): boolean {
    console.log('developing'); // 개발하기
    return true;
  }

  public rest(): boolean {
    console.log('resting'); // 쉬기
    return true;
  }

  public someJob(): boolean {
    console.log('some job 1'); // 뭔가 다른 일
    return true;
  }

  public someJob2(): boolean {
    console.log('some job 2'); // 뭔가 또 다른 일
    return true;
  }
}

const developer = new GoodDeveloper2020();
developer.sleep();
developer.rest();
developer.someJob2();
developer.develop();
developer.eat();
developer.someJob();
```

### LSP: 리스코프 치환 원칙 (Liskov Substitution Principle)

> 만약 S가 T의 서브타입이라면, T는 어떠한 경고도 내지 않으면서, S로 대체가 가능합니다.

![image](/posts/images/solid-principles/lsp-1.png)
![image](/posts/images/solid-principles/lsp-2.jpg)

자식 클래스가 부모클래스의 기능을 똑같이 수행할 수 없을때, 이는 버그를 발생시키는 요인이 됩니다.

만약 어떤 클래스가 자신으로부터 다른 클래스를 생성했다면, 이제 그 클래스는 부모 클래스가 되고, 생성된 클래스는 자식 클래스가 된다. 자식 클래스는 부모 클래스가 할 수 있는 모든 것을 할 수 있어야 한다. 이를 `상속`이라 한다.

자식 클래스는 부모 클래스처럼 똑같은 요청에 대해 똑같은 응답을 할 수 있어야 하고, 응답의 타입 또한 같아야 합니다.

위 그림은 부모 클래스가 커피를 배달하는 모습을 보여줍니다. (커피의 어떤 종류든 가능해야 합니다.) 자식 클래스는 커피의 한 종류인 카푸치노를 배달할 수 있습니다. 하지만 물을 배달하는 것은 허용되지 않습니다.

만약 자식 클래스가 위의 조건을 충족하지 않으면, 자식 클래스가 완전히 변경되어 LSP에 위배된다는 것을 뜻합니다.

>LSP의 목적은 일관성을 유지하여 부모 클래스 또는 자식 클래스를 오류 없이 동일한 방식으로 사용할 수 있도록 하는 것입니다.

#### 예제 코드 (Bad Case)
```typescript
class BadClockParent {
  public date: Date;
  public dateString: string;

  constructor() {
    this.date = new Date();
    this.dateString = this.date.toString();
  }

  public getDate(): string {
    return this.dateString;
  }
}

const clockParent: BadClockParent = new BadClockParent();
clockParent.getDate(); // Thu Jun 18 2020 01:11:27 GMT+0900 (대한민국 표준시)

/**
 * 아래는 자식
 */

class BadClockChild extends BadClockParent {
  constructor() {
    super();
    this.dateString = this.date.toLocaleString();
  }
}

const clockChild: BadClockChild = new BadClockChild();
clockChild.getDate(); // 2020. 6. 18. 오전 1:11:33
```

#### 예제 코드 (Good Case)
```typescript
class GoodClockParent {
  public date: Date;
  public dateString: string;

  constructor() {
    this.date = new Date();
    this.dateString = this.date.toString();
  }

  public getDate(): string {
    return this.dateString;
  }
}

const clockParent: GoodClockParent = new GoodClockParent();
clockParent.getDate(); // Thu Jun 18 2020 01:11:27 GMT+0900 (대한민국 표준시)

/**
 * 아래는 자식
 */

class GoodClockChild extends GoodClockParent {
  public dateLocaleString: string;

  constructor() {
    super();
    this.dateLocaleString = this.date.toLocaleString();
  }

  public getDateWithLocaleString(): string {
    return this.dateLocaleString;
  }
}

const clockChild: GoodClockChild = new GoodClockChild();
clockParent.getDate(); // Thu Jun 18 2020 01:11:27 GMT+0900 (대한민국 표준시)
clockChild.getDateWithLocaleString(); // 2020. 6. 18. 오전 1:11:33
```

### ISP: 인터페이스 분리 원칙 (Interface Segregation Principle)

> 클라이언트는 사용하지 않는 메서드에 대해 의존적이지 않아야 합니다.

![image](/posts/images/solid-principles/isp-1.png)
![image](/posts/images/solid-principles/isp-2.jpg)

클래스가 서로 관계없는 기능들을 가지고 있다면 낭비가 되고, 예상치못한 버그를 발생시킬 수 있습니다.

클래스는 해당 역할에 대한 액션만 수행해야 하고, 이를 제외한 다른 액션은 완전히 삭제하거나 다른 곳(다른 클래스 등)으로 이동시켜야 합니다.

>ISP의 목적은 액션 집합을 더 작은 액션 집합으로 쪼개서, 클래스가 필요한 액션들만 실행할 수 있도록 하는 것 입니다.

#### 예제 코드 (Bad Case)
```typescript
interface IAllInOnePrinter {
  print(): void;
  fax(): void;
  scan(): void;
}

class AllInOnePrinter implements IAllInOnePrinter {
  print() {
    console.log('print');
  }

  fax() {
    console.log('fax');
  }

  scan() {
    console.log('scan');
  }
}
```

#### 예제 코드 (Good Case)
```typescript
interface IPrint {
  print(): void;
}

interface IFax {
  fax(): void;
}

interface IScan {
  scan(): void;
}

class AllInOnePrinter implements IPrint, IFax, IScan {
  print() {
    console.log('print');
  }

  fax() {
    console.log('fax');
  }

  scan() {
    console.log('scan');
  }
}
```

### DIP: 의존성 역전 원칙 (Dependency Inversion)

> 추상(abstraction)은 구체(detail)에 의존하지 않아야 하며, 구체는 추상에 의존적이어야 합니다.<br> 
> 고수준의 모듈은 저수준의 모듈에 의존적이면 안되고, 둘다 추상에 의존적이어야 합니다.

![image](/posts/images/solid-principles/dip-1.png)
![image](/posts/images/solid-principles/dip-2.jpg)

먼저, 쉬운 설명을 위해 용어를 정하도록 합시다.

- 고수준 모듈 (또는 클래스): 도구와 함께 동작하는 클래스.
- 저수준 모듈 (또는 클래스): 수행하기 위한 도구.
- 추상: 두 클래스를 연결하는 인터페이스
- 구체: 도구가 동작하는 방법

DIP는, 액션을 수행할때 클래스가 도구와 융합되면 안된다고 말합니다. 보다 좋은 방법은 인터페이스와 융합하여 클래스와 도구를 연결하는 것 입니다.

두 클래스와 인터페이스는 어떻게 도구가 동작하는지 알 수 없어야 합니다. 하지만, 도구는 인터페이스 사양을 충족해야 합니다.

>DIP의 목적은 인터페이스를 통해 고수준 클래스이 저수준 클래스에 대해 의존성을 가지는 것을 줄이는 것 입니다.

#### 예제 코드 (Bad Case)
```typescript
class HelloWorldOnlyRobot {
  public work() {
    console.log('Hello World!');
  }
}
```

#### 예제 코드 (Good Case)
```typescript
class FlexibleRobot {
  public workType: string;

  constructor(workType: string) {
    this.workType = workType;
  }

  public work() {
    console.log(`I'm doing ${this.workType}.`);
  }
}
```

## 요약
지금까지 우리는 다섯가지 원칙에 대해 살펴보고, 각각의 원칙이 가지는 목표에 대해 알아보았습니다.
이러한 목표들은 여러분의 코드를 조정, 확장, 테스트 하는데에 도움을 줄 것입니다. 아무런 문제도 없이 말이죠!


## 참고자료
- [the-s-o-l-i-d-principles-in-pictures](https://medium.com/backticks-tildes/the-s-o-l-i-d-principles-in-pictures-b34ce2f1e898)<br>
- Clean Architecture (로버트 C. 마틴 지음, 송준이 옮김)
