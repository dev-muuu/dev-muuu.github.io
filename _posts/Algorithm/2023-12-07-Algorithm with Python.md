---
layout: post
title: Algorithm with Python
categories: Algorithm
tags: [Algorithm, Programmers]
---

파이썬으로 알고리즘을 풀기 위해 필요한 개념들을 정리합니다. 

## 문자열
---

- 문자열의 대/소문자화. 

``` python
.upper() # 대문자
.lower() # 소문자
```

- 특정 문자를 기준으로 문자열 분리

``` python
.split(문자)
```

## 배열
---

- 원소 추가

``` python
배열.append(원소)
```

- 마지막 원소 삭제

``` python
배열.pop()
```

- 정렬

``` python
배열.sort() # 오름차순
배열.sort(reversed=True) # 내림차순
```


- map으로 배열 값 변환하기

``` python
list(map(int, 배열)) # str을 int로 변환
ex. list(map(int, s))
```

## 딕셔너리
---

- 빈 딕셔너리 선언.

``` python
cache = {}
```

- 딕셔너리에 값이 존재하는지 체크.

``` python
if 키 in 딕셔너리
```

- 최소값을 가지는 키 값을 반환

``` python
min(딕셔너리, key=딕셔너리.get)
```

- 딕셔너리에서 키 값을 제거

``` python
del 딕셔너리[키 값]
```
