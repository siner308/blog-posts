---
title:  "JAVA의 컴파일 과정"
subtitle: "JIT의 동작원리와 코틀린에서의 컴파일 차이점"
tags:
    - java
date:   2021-12-25
image: https://user-images.githubusercontent.com/34048253/147363191-49b62b21-ef59-422d-a58a-341dbe124190.png
---

JIT(Just-In-Time)은 프로그램을 실제 실행하는 시점에 기계어로 번역하는 컴파일 기법이다.
자바의 경우 자바 컴파일러가 자바 프로그램 코드를 바이트코드로 변환한 다음, 실제 바이트코드를 실행하는 시점에서 자바 가상 머신이 바이트코드를 JIT 컴파일을 통해 기계어로 변환한다.

# 작동 방식
JAVA 소스코드(.java)를 바이트코드(.class)로 변환하는 작업은 javac 컴파일러의 도움으로 진행된다.
이후 .class 파일은 런타임에 JVM에 의해 로드되고, 인터프리터의 도움으로 기계가 이해할 수 있는 코드로 변환된다.
JIT 컴파일러는 JVM의 일부이다. JIT 컴파일러가 활성화되면, JVM은 .class파일의 메서드 호출을 분석하여 보다 효율적인 원시코드로 변환한다. 또한 우선순위가 지정된 메서드 호출이 최적화되도록 한다.
이렇게 컴파일 된 코드는 캐시처리 되어 이후 같은 코드에 대해서는 컴파일을 하지 않고 직접 실행하게 되고, 이를 통해 실행의 성능과 속도를 증가시킬 수 있다. 

![image](https://user-images.githubusercontent.com/34048253/147363191-49b62b21-ef59-422d-a58a-341dbe124190.png)


# 코틀린에서의 컴파일
자바에서 .java -> .class 의 과정을 거치듯이
코틀린에서도 .kt 파일을 .class로 변환해주기만 하면 JVM에서는 동일하게 사용이 가능하다.  

<img src="https://user-images.githubusercontent.com/34048253/147363062-b8395f1c-d4cf-46c6-8fbd-5295ded706f4.png" width=250 />

# references
- [Java Basic Interview Questions](https://www.interviewbit.com/java-interview-questions/)
- [JIT 컴파일](https://ko.wikipedia.org/wiki/JIT_%EC%BB%B4%ED%8C%8C%EC%9D%BC)
