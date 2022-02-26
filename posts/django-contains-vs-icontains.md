---
title: "[TIL] django contains vs icontains"
subtitle: "TIL django query"
tags:
    - django
date: 2021-10-30
image: https://user-images.githubusercontent.com/34048253/155850024-c71c1bea-800c-4ae5-a4ec-6f68c5584086.png
---

||contains|icontains|
|---|---|---|
|의미|대소문자를 구별한다|대소문자를 구별하지 않는다|
|SQL|'LIKE BINARY %s'|'LIKE %s'|

![image](https://user-images.githubusercontent.com/34048253/155849989-d334fa27-572c-4ca2-85d3-32e0042bdfef.png)


# 출처
- [https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-contains](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-contains)
