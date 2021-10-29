---
layout: post
title:  "django contains vs icontains"
subtitle: "TIL django query"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - django
date:   2021-10-30
multilingual: flase
---


||contains|icontains|
|---|---|---|
|의미|대소문자를 구별한다|대소문자를 구별하지 않는다|
|SQL|'LIKE BINARY %s'|'LIKE %s'|
