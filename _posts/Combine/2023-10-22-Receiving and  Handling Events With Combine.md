---
layout: post
title: Receiving and Handling Events with Combine
categories: Combine
tags: [iOS, Combine, Document]
---

## [Apple Documentation](https://developer.apple.com/documentation/combine/receiving-and-handling-events-with-combine)

## Overview
---
Combine 프레임워크는 앱이 이벤트를 처리하는 방법에 대한 선언적 접근 방식을 제공한다. 여러 delegate 패턴 또는 completion handler를 구현하는 대신, 주어진 이벤트에 대한 단일 처리 체인을 생성할 수 있다. 체인의 각 부분은 이전 단계에서 수신한 value에 대해 개별적 동작을 수행하는 Combine 연산자이다.

## Connect a Publisher to a Subscriber
---
Combine으로 부터 text field의 notification을 받기 위해서는 NotificationCenter의 default 인스턴스와 `publisher(for: object:)`를 호출해야 한다. subscriber는 assoicated type인 Input, Output, Failure를 정의해야 한다. Failure는 publisher가 생산하고 subscriber가 받게 되는 error의 종류를 나타낸다. Combine은 publisher에 output과 failure 타입들을 자동으로 매치시켜주는 2가지의 built-in subscriber를 제공한다. 

1. `sink(receiveCompletion:receiveValue:)`는 2가지의 closure를 가진다. 정상적으로 publisher가 종료되거나 error로 인해 실패했을 때 Subscribers.Completion이 방출되며 receiveCompletion가 동작하게 된다.
2. `assign(to: on:)`는 모든 element를 지정된 object의 속성에 즉시 할당한다. 

위의 2가지 구독자 모두 publisher에게 무제한의 element를 요청한다. element를 받는 속도를 제어하고 싶다면, Subsctiber protocol를 활용해 자신만의 subscriber를 구현하면 된다. 

``` swift
//iOS
 NotificationCenter.default
    .publisher(for: UITextField.textDidChangeNotification, object: textfield)
    .sink(receiveCompletion: { print ($0) },
          receiveValue: { print ($0) })
```


## Change the Output Type with Operators
---
위의 섹션에서는 `sink(receiveValue:)`의 receiveValue closure에서 기본적으로 element에 대한 모든 작업을 수행했다. 만약 element에 대해서 많은 커스텀 작업 등을 모두 closure에서 수행하게 될 경우 부담이 될 수 있다. Combine은 operator 결합을 통해 event를 커스텀하며 전달할 수 있도록 한다. 

+ `map(_:)`
+ `flatMap(maxPublishers:_:)`
+ `reduce(_:_:)`
 

operator를 통해 publisher 체인이 원하는 결과값으로 변환되면, `sink(receiveCompletion:receiveValue:)` subscriber 메서드를 `assign(to: on:)` 메서드로 대신 적용할 수 있다. 이 메서드를 통해 view model 인스턴스의 적절한 프로퍼티에 직접 바인딩 시켜줄 수 있다. 

``` swift
let sub = NotificationCenter.default
    .publisher(for: NSControl.textDidChangeNotification, object: filterField)
    .map( { ($0.object as! NSTextField).stringValue } )
    .assign(to: \MyViewModel.filterString, on: myViewModel)
```

## Customize Publishers with Operators 
---
수동으로 operator를 추가하는 방식을 활용해 publisher 인스턴스를 확장할 수 있다. operator를 사용해 이벤트 처리 체인을 개선할 수 있는 방식으로 3가지 방식이 존재한다. 

+ `filter(_:)`를 활용하여 특정 조건에 부합하는 element만 receive 할 수 있다. 
+ 대용량 데이터베이스를 쿼리하는 경우와 같이 필터링 작업에 비용이 많이 드는 경우, 사용자가 입력을 중지할 때까지 기다려야 할 수 있다. 이 경우에 `debounce(for:scheduler:options:)` operator를 사용하면 이벤트를 전송하기 전에 경과해야 하는 최소 기간을 설정할 수 있다. RunLoop 클래스는 시간 지연을 초/밀리초 단위로 지정할 수 있는 편리함을 제공한다. 
+ 만약 결과값이 UI를 업데이트한다면, `receive(on:options:)` 호출을 통해 main thread에서 진행되도록 설정할 수 있다. 

``` swift
let sub = NotificationCenter.default
    .publisher(for: NSControl.textDidChangeNotification, object: filterField)
    .map( { ($0.object as! NSTextField).stringValue } )
    .filter( { $0.unicodeScalars.allSatisfy({CharacterSet.alphanumerics.contains($0)}) } )
    .debounce(for: .milliseconds(500), scheduler: RunLoop.main)
    .receive(on: RunLoop.main)
    .assign(to:\MyViewModel.filterString, on: myViewModel)
```

## Cancel Publishing when Desired
---
publisher는 정상적으로 완료되거나 실패할 때까지 element를 계속 방출한다.만약 더 이상 subscribe를 하고 싶지 않다면 subscribe를 취소할 수 있다. `sink(receiveComplete: receiveValue:)` 및 `assign(to: on:)`에서 생성된 subscirber 타입은 모두 `cancel()` 메서드를 제공하는 Cancelable 프로토콜을 구현한다. 반대로 커스텀 subscriber를 생성할 때는 subscribe를 취소할 수 있도록 Cancelable 프로토콜을 채택해야 한다.
