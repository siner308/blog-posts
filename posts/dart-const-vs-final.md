---
title: "[Dart Programming] final과 const의 차이점"
subtitle: "상수를 표현하는 두가지 방법"
tags:
    - dart
date: 2020-12-08
---

final과 const는 모두 상수(constant)를 선언할 때 사용되는 키워드입니다.
Dart에서는 final과 const를 통해 선언된 변수는 수정이 불가능하도록 막아줍니다.

const는 compile 단계에서 값이 정해지는 상수입니다. 그렇기 때문에 매번 값이 바뀌는 `random`이나 `DateTime.now()`와 같이 멱등이 보장되지 않는 값들은 const로 선언할 수 없습니다.

### const를 사용한 방식

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    const wordPair = WordPair.random();
    return MaterialApp(
      title: 'Welcome to Flutter',
      home: Scaffold(
        appBar: AppBar(
          title: Text('Welcome to Flutter'),
        ),
        body: Center(
          child: Text(wordPair.asPascalCase),
        ),
      ),
    );
  }
}
```

### const 사용시 나타나는 에러
```bash
lib/main.dart:9:31: Error: Cannot invoke a non-'const' factory where a const expression is expected.
Try using a constructor or factory that is 'const'.
    const wordPair = WordPair.random();
                              ^^^^^^
```

### final를 사용한 방식

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final wordPair = WordPair.random();
    return MaterialApp(
      title: 'Welcome to Flutter',
      home: Scaffold(
        appBar: AppBar(
          title: Text('Welcome to Flutter'),
        ),
        body: Center(
          child: Text(wordPair.asPascalCase),
        ),
      ),
    );
  }
}
```

### 결과
<img src="https://user-images.githubusercontent.com/34048253/101371941-af56b800-38ee-11eb-90b5-ff98b11cb4cb.png" width="400" >

### 출처
- [https://flutter.dev/docs/get-started/codelab#step-2-use-an-external-package](https://flutter.dev/docs/get-started/codelab#step-2-use-an-external-package)
- [https://www.tutorialspoint.com/dart_programming/dart_programming_variables.htm](https://www.tutorialspoint.com/dart_programming/dart_programming_variables.htm)
- [https://medium.com/dartlang-korea/dart-final-%EA%B3%BC-const-bc8c6c024ef4](https://medium.com/dartlang-korea/dart-final-%EA%B3%BC-const-bc8c6c024ef4)
