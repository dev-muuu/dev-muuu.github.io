---
layout: post
title: Combine
categories: Combine
tags: [iOS, Combine, Document]
---

## [Apple Documentation](https://developer.apple.com/documentation/combine)

## Overview
---
Combine은 시간이 경과함에 따라 값이 변화하는 value를 제공하기 위한 Swift API이며, value에 대해 비동기 이벤트로 처리할 수 있도록 도와주는 프레임워크이다.Combine에서 publisher는 항상 변화하는 값을 노출시키기 위해 선언하며, subscriber는 publisher로 부터 노출되는 값을 받기 위해 선언한다. 

- Publisher 프로토콜은 시간이 경과함에 따라 value를 순차적으로 전달할 수 있는 타입에 선언한다. 
- Publisher는 명시적으로 subscriber가 하는 일이 지정된 경우에만 value를 방출할 수 있다.  

Combine을 사용하면, '중첩 closure'나 '협약 기반 callback'과 같은 골치 아픈 기술 제거를 할 수 있어 코드를 쉽게 작성하고 유지보수할 수 있다. 

## Topics
---
#### [Receiving and Handling Events with Combine]({{ /2023-10-22-Receiving%20and%20%20Handling%20Events%20With%20Combine.md | relative_url }})
