---
layout: post
title: use Combine with UITextField
categories: Combine
tags: [iOS, Combine]
---

## Overview

UITextField를 Combine으로 사용하기 위해서는 NotificationCenter을 활용해야 한다. `extension`을 활용하면 UITextField에 대한 event를 조금 더 편리하게 받을 수 있다. 


## 1. default
---

가장 기본적인 방식이다. NotificationCenter 인스턴스에 직접 접근하며 어떤 event를 처리받을 것인지 지정해주면 된다. NotificationCenter에서 데이터를 얻기 위해서는 타입 캐스팅 작업을 거쳐야 한다.

``` swift
NotificationCenter.default
    .publisher(for: UITextField.textDidChangeNotification, object: textfield)
    .compactMap{
        $0.object as? UITextField
    }.map{
        $0.text ?? ""
    }.sink(receiveValue: {
       print($0)
    })
    .store(in: &cancellable)
}
```

## 2. convenience 1
---

섹션 1 방식의 경우 UITextField로 부터 event를 받으려면 매번 NotificationCenter에 접근하고 타입 캐스팅 작업을 거쳐야 한다는 번거로움이 있다. UITextField를 `extension`하여 연산 프로퍼티를 추가하면 문제점을 개선할 수 있다. 

``` swift
extension UITextField {
    var publisher: AnyPublisher<String, Never> {
        return NotificationCenter.default
            .publisher(for: UITextField.textDidChangeNotification, object: self)
            .compactMap{
                $0.object as? UITextField
            }.map{
                $0.text ?? ""
            }
            .eraseToAnyPublisher()
    }
}
```

아래와 같이 사용할 수 있다. 

``` swift
textfield.publisher
    .sink(receiveValue: {
        print($0)        
    })            
    .store(in: &cancellable)
```

## 3. convenience 2.
---

섹션 2 방식은 `UITextField.textDidChangeNotification` 이벤트에 대해서만 다룰 수 있으며, 반드시 text에 대해서만 element를 방출한다는 문제점이 있다. 

RxSwift는 textfield에 대해서 아래와 같이 text 값에 대해서 접근한다.

```
textfield.rx.text
textfield.rx.text.orEmpty
```

값 변경 뿐만이 아니라 다양한 이벤트를 받을 수 있도록 하며, RxSwift와 같이 `text`프로퍼티에 접근해 text 데이터를 얻는 방식으로 수정해보도록 하겠다. 

``` swift
extension UITextField {
    func publisher(for event: UITextField.Event) -> AnyPublisher<UITextField, Never> {
        return NotificationCenter.default
                .publisher(for: convertEvent(), object: self)
                .compactMap{
                    $0.object as? UITextField
                }
                .eraseToAnyPublisher()
            
        func convertEvent() -> NSNotification.Name {
            switch event {
            case .valueChanged:     return UITextField.textDidChangeNotification
            default:                fatalError()
            }        
        }
    }
}

extension AnyPublisher where Self == AnyPublisher<UITextField, Never>{
    var text: AnyPublisher<String, Never>{
        self.map{
            $0.text ?? ""
        }
        .eraseToAnyPublisher()
    }
}
```
아래와 같이 사용할 수 있다. 

``` swift 
textfield.publisher(for: .valueChanged)
    .text
    .sink(receiveValue: {
        print($0)
    })
    .store(in: &cancellable)
```

## 4. with View Model
---

섹션 3까지는 view controller에서 text field에 접근하고 값을 처리하는 방식이었다. mvvm 아키텍처 적용으로 view model에서 text 데이터를 처리해야 하는 경우 2가지 방식을 사용할 수 있다. 

#### 1. View Controller의 Publisher와 View Model의 Publisher
---

첫 번째 방식은 먼저 view controller에서 text element를 수신하면 이를 `assign(to: on:)`을 통해 view model의 publisher가 구독하고, 이를 다시 재방출하는 로직이다.   
하지만 이 방식은 2개의 Publisher를 사용하게 되어 메모리 측면에서 비효율적이라는 판단을 하게 되었다. 

``` swift
final class DefaultSinkTextFieldViewModel{
    
    @Published var text: String = ""
    private var cancellable: Set<AnyCancellable> = []
    
    init(){ 
        bind()
    }
    
    func bind(){
        $text.sink(receiveValue: {
            print($0)
        })
        .store(in: &cancellable)
    }
}
```

``` swift 
private func bind(){
     textfield.publisher(for: .valueChanged)
        .text
        .assign(to: \.text, on: viewModel)
        .store(in: &cancellable)
}
```

#### 2. View Controller의 Publisher, View Model로 전달
---

2 번째 방식은 view controller의 publisher를 view model이 처리하도록 넘겨주는 방식이다. 첫 번째 방식과 다르게 publisher 1 개만 사용한다.

``` swift 
protocol SinkTextFieldViewModel {
    func bind(input: SinkTextFieldViewModelInput)
}

protocol SinkTextFieldViewModelInput {
    var text: AnyPublisher<String, Never> { get }
}
```

``` swift
struct DefaultSinkTextFieldViewModelInput: SinkTextFieldViewModelInput {
    let text: AnyPublisher<String, Never>
}

final class DefaultSinkTextFieldViewModel: SinkTextFieldViewModel{
    
    private var cancellable: Set<AnyCancellable> = []
    
    init(){ }
    
    func bind(input: SinkTextFieldViewModelInput) {
        input.text
            .sink(receiveValue: {
                print($0)
            })
            .store(in: &cancellable)
    }
}
```

``` swift
viewModel.bind(
    input: DefaultSinkTextFieldViewModelInput(
        text: textfield.publisher(for: .valueChanged).text
    )
)
```