---
layout: post
title:  "싱글톤 패턴 (Singleton Pattern)"
subtitle: "반복되는 인스턴스 낭비를 줄이자"
author: "Siner"
header-img: "img/post_headers/2020-02-23-singleton-pattern.png"
catalog: true
header-mask:  0.3
tags:
    - HTTP
    - Architecture
date:   2020-02-23
multilingual: false
---

참고자료
* [Singleton pattern](https://en.wikipedia.org/wiki/Singleton_pattern)
* [타입스크립트 디자인 패턴](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788960779914&orderClick=LAG&Kc=)

---

## 1. Intro
특정 클래스의 인스턴스가 단 하나만 존재해야 하는 경우에 싱글톤 패턴을 사용합니다.

>싱글톤 패턴은, 반복적인 디자인 문제를 해결하는 방법을 설명하는 책인 [Design Patterns](http://wiki.c2.com/?DesignPatternsBook)의 23가지 디자인 패턴 중 하나입니다.<br>
>이 책에서는 반복되는 디자인 문제를 어떻게 유연하고 재사용 가능하도록 디자인하는지 서술하고 있습니다.<br>
>싱글톤 패턴은, 클래스의 인스턴스를 시스템 내에 단 한개만 존재하도록 제한시키는 소프트웨어 디자인 패턴입니다. 시스템 전체에서 작업을 조정하는 데 정확히 하나의 개체가 필요한 경우에 유용합니다.<br>

## 2. Problem
싱글톤 패턴은 다음과 같은 문제를 해결할 수 있습니다.
- 클래스의 인스턴스를 단 하나만 생성하고 싶은 경우, 이것을 어떻게 보장할 수 있는가?
- 클래스의 단일 인스턴스에 쉽게 접근할 수 있는 방법은 무엇일까?
- 클래스는 인스턴스화를 어떻게 제어할 수 있는가?
- 어떻게 클래스의 인스턴스 수를 제한할 수 있는가?

## 3. Solve
싱글톤 패턴은 위의 문제를 다음과 같이 해결합니다.
- 클래스의 생성자(constructor)를 숨긴다.
- getInstance()라는 public static 함수를 만들어, 단일 인스턴스를 반환하도록 한다.

이 패턴의 핵심 아이디어는 클래스 자체가 인스턴스화 제어 (한 번만 인스턴스화 됨)를 책임지게 하는 것입니다.
숨겨진 생성자(private)는 클래스 외부에서 인스턴스를 생성할 수 없다는 것을 증명합니다.
클래스 이름과 작업 이름 (Singleton.getInstance())을 사용하여 쉽게 액세스 할 수 있습니다.


## 4. Examples

#### Typescript
```typescript
class Singleton {
    private static _default: Singleton;
    static getInstance(): Singleton {
        if (!Singleton._default) {
            Singleton._default = new Singleton();
        }
        return Singleton._default;
    }
}
```

#### Java
```java
public final class Singleton {

    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

## 5. Lazy initialization
싱글톤 인스턴스가 멀티스레드 환경에서 동시에 호출된다면, 두개 이상의 인스턴스가 생성될 수 있습니다.<br>
아래의 예시는 이를 방지하기 위한 thread-safe 샘플코드입니다. double-checked locking을 사용합니다.

#### Java
```java
public final class Singleton {

    private static volatile Singleton instance = null;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }

        return instance;
    }
}
```
