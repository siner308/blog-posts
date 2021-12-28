---
title:  "JAVA 자료구조 정리"
subtitle: "java data structure cheat sheet"
tags:
    - java
date:   2021-12-29
---

# String
String은 리터럴로 표기가 가능하지만 primitive 자료형은 아니다. String은 리터럴 표현식을 사용할 수 있도록 자바에서 특별 대우 해 주는 자료형이다.

`String a = "happy java"` 와 `String a = new String("happy java")` 는 같은 값을 갖게 되지만 완전히 동일하지는 않다. 첫번째 방식을 리터럴(literal) 표기라고 하는데 객체 생성없이 고정된 값을 그대로 대입하는 방법을 말한다. 위 예에서 리터럴 표기법은 "happy java" 라는 문자열을 intern pool 이라는 곳에 저장하고 다음에 다시 동일한 문자열이 선언될때는 cache 된 문자열을 리턴한다. 두번째 방식은 항상 새로운 String 객체를 만든다.

문자열의 값을 비교할때는 반드시 equals 를 사용해야 한다. == 연산자를 사용할 경우 다음과 같은 경우가 발생할 수 있다.

```java
String a = "hello";
String b = new String("hello");
System.out.println(a.equals(b));  // true
System.out.println(a==b);  // false
```

# StringBuffer
insert 메소드는 특정 위치에 원하는 문자열을 삽입할 수 있다.

```java
StringBuffer sb = new StringBuffer();
sb.append("jump to java");
sb.insert(0, "hello ");
System.out.println(sb.toString());
```

# StringBuffer.append vs String + String

아래의 두 예시 모두 `hello jump to java`를 출력하지만 내부적으로 객체가 생성되고 메모리가 사용되는 과정은 다르다.

첫번 째 예제의 경우 StringBuffer 객체는 한번만 생성된다. 두번 째 예제는 String 자료형에 `+` 연산이 있을 때마다 새로운 String 객체가 생성된다(문자열 간 `+` 연산이 있는 경우 자바는 자동으로 새로운 String 객체를 만들어 낸다). 두번 째 예제에서는 총 4개의 String 자료형 객체가 만들어지게 된다.

```java
StringBuffer sb = new StringBuffer();  // StringBuffer 객체 sb 생성
sb.append("hello");
sb.append(" ");
sb.append("jump to java");
String result = sb.toString();
System.out.println(result);
```

```java
String result = "";
result += "hello";
result += " ";
result += "jump to java";
System.out.println(result);
```

# Array
사실 python과 js와 같은 언어에서는 동적으로 길이가 변하는 List를 배열이라고 부르고 있었다.
java에서는 Array와 List가 모두 자료형으로 존재한다.
고정된 길이의 `배열`을 사용해야 할 때에는 Array를(아래의 예시) 사용하자.

```java
String[] weeks = new String[7];
weeks[0] = "월";
weeks[1] = "화";
weeks[2] = "수";
weeks[3] = "목";
weeks[4] = "금";
weeks[5] = "토";
weeks[6] = "일";
```

# ArrayList
ArrayList를 사용하기 위해서는 아래와 같이 import를 해줘야한다.

```java
import java.util.ArrayList;
```

## method
```java
arr.get(2); // 두번째 요소에 접근
arr.size(); // ArrList의 갯수
arr.contains("123"); // 포함여부. return boolean
arr.remove("123"); // 값들 중 "123"인 항목을 삭제하고 결과를 true,false로 리턴
arr.remove(0); // 0번 인덱스를 삭제하고 삭제된 항목 리턴
```

## join
```java
import java.util.ArrayList;
import java.util.Arrays;

ArrayList<String> arr1 = new ArrayList<>(Arrays.asList("1", "2", "3"));
String joined1 = String.join(",", arr1);

String[] arr2 = new String[]{ "1", "2", "3" };
String joined2 = String.join(",", arr2);
```

## order
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;

ArrayList<String> arr = new ArrayList<>(Arrays.asList("1", "2", "3"));
arr.sort((Comparator.naturalOrder()); // Comparator.reverseOrder()
```

## 생성
```java
import java.util.ArrayList;
import java.util.Arrays;

// way 1
ArrayList<String> arr1 = new ArrayList<>();
arr1.add("1");
arr1.add("2");

// way2
String[] data = { "1", "2" };
ArrayList<String> arr2 = new ArrayList<>(Arrays.asList(data));
```

# HashMap
```java
// put, get, size, containsKey, remove
import java.util.HashMap;
HashMap<String, String> map = new HashMap<>();
map.put("key", "value");
map.get("key"); // value
map.size(); // 1
map.containsKey("key"); // true
map.remove("key"); // "value"
```

```java
// keySet
import java.util.HashMap;
HashMap<String, String> map = new HashMap<>();
map.put("key1", "value1");
map.put("key2", "value2");
map.keySet(); // [key1, key2]
```

- LinkedHashMap: 입력된 순서대로 데이터를 저장한다
- TreeMap: key의 오름차순 순서로 데이터를 저장한다

# HashSet
```java
import java.util.HashSet;

HashSet<String> set = new HashSet<>(Arrays.asList("h", "e", "l", "l", "o")); // [e, h, l ,o]
```

```java
// retainAll, addAll, removeAll
import java.util.HashSet;
import java.util.Arrays;

HashSet<Integer> s1 = new HashSet<>(Arrays.asList(1,2,3));
HashSet<Integer> s2 = new HashSet<>(Arrays.asList(2,3,4));

new HashSet<>(s1).retainAll(s2); // [2, 3] 교집합
new HashSet<>(s1).addAll(s2); // [1, 2, 3, 4] 합집합
new HashSet<>(s1).removeAll(s2); // [1] 차집합
```

```java
// add, addAll
import java.util.HashSet;
import java.util.Arrays;

HashSet<String> set = new HashSet<>();
set.add("1");
set.addAll(Arrays.asList("2", "3");
```

# enum
```java
enum Language {
  ENGLISH,
  KOREAN,
  JAVA
};
```

```java
for (Language language: Language.values()) {
  // ...
}
```
