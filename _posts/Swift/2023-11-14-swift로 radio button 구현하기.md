---
layout: post
title: swift로 radio button 구현하기
categories: Swift
tags: [iOS, Swift]
---

## 1. 구상
---

`radio button` ui를 살펴보면 보통 아래와 같다. 

![Github_Logo](https://cdn-images-1.medium.com/max/521/1*mAU3lQVM6mdr1mHfvaRkeg.png)  

이를 통해 필요한 것이 무엇인지 생각해보면 아래와 같다. 

- cell을 표현할 ui
- cell에 각 케이스를 나타낼 title
- cell이 선택되었을 때와 선택되지 않았을 때를 나타낼 ui

## 2. protocol 설계
---

### Cell
---

`title label` 만 필수로 구현하도록 설정하였으며, 이외 버튼과 같은 다른 component가 필요할 경우 구현체에서 자유롭게 추가해주면 된다. 

``` swift
public protocol RadioButtonCell where Self: UIView {
    var titleLabel: UILabel { get set }
    func select() //선택되었을 때의 UI 구현
    func deselect() //선택되지 않았을 때의 UI 구현
}
```

### Data
---

`RadioButtonCell`과 마찬가지로 `title` 데이터만 필수로 지정하였다. `enum` 타입으로 데이터를 생성하고, `RadioButtonData` 프로토콜을 채택하면 된다. 

``` swift
public protocol RadioButtonData {
    var title: String { get }
}
```

## 3. radio button class 구현
--- 

spacing과 axis 지정의 편리함과 `radio button`은 많지 않은 케이스를 다룰 가능성이 높기에 view 추가의 편의를 위해 `UIStackView` 상속을 선택하였다. 

각 케이스의 view를 배열에 저장하여 인덱스로 접근하는 것이 아니라 `tag`를 활용하여 선택된 view를 가지고 올 수 있도록 구현하였다. 

``` swift
public final class RadioButtonView<T: RadioButtonCell>: UIStackView {

    public init(elements: [any RadioButtonData], cellType: T.Type) {
        self.data = elements
        super.init(frame: .zero)
        hierarchy()
        initialize()

    }
    
    required init(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    //MARK: Public property
    public var elementPublisher: AnyPublisher<any RadioButtonData, Never> { //새롭게 선택된 element를 방출.
        $selectedView
            .map{ self.data[self.convert(tag: $0!.tag)] }
            .eraseToAnyPublisher()
    }
    
    public var currentElement: (any RadioButtonData) {
        data[convert(tag: selectedView!.tag)]
    }
    
    //MARK: Private property
    private typealias Index = Int
    private typealias Tag = Int
    
    private let data: [any RadioButtonData]
    
    @Published private var selectedView: RadioButtonCell? {
        didSet {
            oldValue?.deselect()
            selectedView?.select()
        }
    }
    
    private func convert(tag: Int) -> Index {
        tag-1
    }
    
    private func convert(index: Int) -> Tag {
        index+1
    }

    private func hierarchy(){
        for (i, element) in data.enumerated() {
            let elementView = T()
            setDefault()
            addTarget()
            addArrangedSubview(elementView)
            
            func setDefault() {
                elementView.tag = convert(index: i)
                elementView.titleLabel.text = element.title
                elementView.deselect()
            }
            func addTarget(){
                elementView.addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(addElementTarget)))
            }
        }
    }
    
    @objc private func addElementTarget(_ recognizer: UITapGestureRecognizer) {
        guard let view = recognizer.view as? RadioButtonCell else { return }
        selectedView = view
    }
    
    private func initialize() {
        let initIndex = 0
        guard let initView = viewWithTag(convert(index: initIndex)) as? RadioButtonCell else { return }
        selectedView = initView
    }
}
```

## 4. 사용
---

``` swift 
enum Gender: String, CaseIterable, RadioButtonData {
    
    case man
    case woman
    
    var title: String {
        rawValue
    }
}
````

``` swift 
final class RadioButtonViewController: ViewController {

    private let radioButtonFrame: RadioButtonView = {
        let frame = RadioButtonView(elements: Gender.allCases, cellType: DefaultRadioButtonCell.self)
        frame.axis = .vertical
        frame.spacing = 10
        return frame
    }()

    ...
    
    override func bind() {
        radioButtonFrame.elementPublisher
            .sink(receiveValue: {
                print($0)
            })
            .store(in: &cancellable)
    }
}
```
