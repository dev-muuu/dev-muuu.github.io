---
layout: post
title: Processing URL session data task results with Combine
categories: Combine
tags: [iOS, Combine, Document]
---

## [Apple Documentation](https://developer.apple.com/documentation/foundation/urlsession/processing_url_session_data_task_results_with_combine)

## Overview
---

`URLSession`을 통한 작업 수행은 비동기로 동작한다. 네트워크 엔드 포인트, 파일 시스템 및 URL 기반 소스에서 데이터를 가져오려면 시간이 걸린다. URL 로딩 시스템은 delegate 또는 completion handler에게 비동기로 결과를 전달한다. Combine 프레임워크도 비동기로 처리하며, 이를 사용하여 URL 작업을 처리하면 코드가 단순화되며 강화된다. 


## Create a data task publisher
---

`URLSession`은 Combine publisher인 `URLSession.DataTaskPubisher`를 제공한다. 이는 `URL`이나 `URLRequest`로 부터 받은 데이터를 결과값으로 publish 한다. `dataTaskPublisher(for:)`을 통해 이 publisher를 생성할 수 있다. task가 완료되면 아래 2 가지를 방출한다. 
- task가 성공할 경우, 데이터와 URLResponse를 튜플에 담아 반환한다
- task가 실패할 경우, 에러를 반환한다. 

`dataTask(with:completionHandler:)`로 전달되는 completion handler와 다르게, publisher가 이미 `data`나 `error`의 랩핑을 해제하기 때문에 결과로 전달 받는 타입은 optional 타입이 아니다. 

URLSession의 completion handler 기반 코드를 사용할 때는 handler closure에서 에러 작업, 데이터 등 모든 작업을 수행해야 한다. 하지만 data task publisher를 사용하면, 이러한 책임들은 Combine의 operator들에게 넘길 수 있다. 


## Convert incoming raw data to your types with Combine operators
---

data task가 성공적으로 완료되면, raw Data를 URLResponse와 함께 튜플에 담아 앱에 전달해준다. 대부분 raw 상태인 Data를 적절한 타입으로 변환해줘야 한다. Combine은 이러한 변환을 연달아 체인처럼 수행할 수 있는 operator들을 제공한다. 

- `map(_:)` operator를 통해 튜플의 내용을 다른 타입으로 변환할 수 있다. 
- `tryMap(_:)` operator를 통해 데이터를 검사하기 전에 response를 검사할 수 있으며, 이때 response가 unacceptable하다면 에러를 던지면 된다. 
- `decode(type:decoder:)` operator를 통해 raw Data를 Decodable을 채택한 타입으로 쉽게 변환할 수 있다.

``` swift
struct User: Codable {
    let name: String
    let userID: String
}

let url = URL(string: "https://example.com/endpoint")!
cancellable = urlSession
    .dataTaskPublisher(for: url)
    .tryMap() { element -> Data in
        guard let httpResponse = element.response as? HTTPURLResponse,
            httpResponse.statusCode == 200 else {
                throw URLError(.badServerResponse)
            }
        return element.data
    }
    .decode(type: User.self, decoder: JSONDecoder())
    .sink(receiveCompletion: { print ("Received completion: \($0).") },
          receiveValue: { user in print ("Received user: \(user).")})

```

## Retry transient errors and catch and replace persistent errors
---

네트워크를 사용하는 모든 앱은 오류가 발생할 수 있음을 인지하고, 에러를 잘 다룰 수 있어야 한다. 일시적인 네트워크 오류는 매우 흔하므로 실패한 data task를 즉시 재시도할 수 있다. `URLSession`의 completion handler 관용구를 사용하면, 재요청을 위해 완전히 새로운 task를 만들어야 한다. 하지만 Combine을 활용하면 `retry(_:)` operator로 쉽게 처리가 가능하다. 이 operator는 특정 횟수만큼의 upstream publisher에 대한 구독을 만들어 오류를 처리한다. 이때 네트워크 작업은 비용이 많이 들기 때문에 몇 번만 재시도하고 멱등(결과가 달라지지 않는지) 인지 확인한다. 

Combine operator 중에는 에러가 그대로 subscriber에게 전달되지 않고 에러를 대체할 수 있다. 

- `catch(_:)`는 오류를 다른 publisher로 대체한다. fallback URL로 부터 데이터를 로드하는 publisher와 같이 다른 URLSession.DataTaskPublisher와 함께 사용할 수 있다.
- `replaceError(with:)`는 오류를 제공하는 element로 오류를 대체한다. 응용 프로그램에서 의미가 있다면, 이를 사용하여 URL에서 로드할 것으로 예상되는 값을 대체할 수 있다. 

``` swift
let pub = urlSession
    .dataTaskPublisher(for: url)
    .retry(1)
    .catch() { _ in
        self.fallbackUrlSession.dataTaskPublisher(for: fallbackURL)
    }

cancellable = pub
    .sink(receiveCompletion: { print("Received completion: \($0).") },
          receiveValue: { print("Received data: \($0.data).") })
```

## Move work between dispatch queues with scheduling operators
---

`URLSession`의 delegate와 completion handler를 사용할 때, 기본적으로 세션은 고정된 delegateQueue에서 코드가 다시 호출된다. 특정 dispatch queue에서 callback을 실행시키기 위해서는 `receive(on:)`을 사용해 수동으로 설정해주면 된다.

``` swift
cancellable = urlSession
    .dataTaskPublisher(for: url)
    .receive(on: DispatchQueue.main)
    .sink(receiveCompletion: { print ("Received completion: \($0).") },
          receiveValue: { print ("Received data: \($0.data).")})
```


## Share the result of a data task publisher with multiple subscribers
---

네트워크 요청은 많은 비용이 들기 때문에 불필요하게 요청을 반복하면 안된다. Combine은 단일 `URLSession.DataTaskPublisher`에 대해 여러 downstream subscriber를 지원한다. `share()` operator를 사용하면 되며, 이 operator을 통해 다양한 operator 체인과 subscriber들을 연결시킬 수 있다. 

``` swift
let sharedPublisher = urlSession
    .dataTaskPublisher(for: url)
    .share()


cancellable1 = sharedPublisher
    .tryMap() {
        guard $0.data.count > 0 else { throw URLError(.zeroByteResource) }
        return $0.data
    }
    .decode(type: User.self, decoder: JSONDecoder())
    .receive(on: DispatchQueue.main)
    .sink(receiveCompletion: { print ("Received completion 1: \($0).") },
          receiveValue: { print ("Received id: \($0.userID).")})


cancellable2 = sharedPublisher
    .map() {
        $0.response
    }
    .sink(receiveCompletion: { print ("Received completion 2: \($0).") },
           receiveValue: { response in
            if let httpResponse = response as? HTTPURLResponse {
                print ("Received HTTP status: \(httpResponse.statusCode).")
            } else {
                print ("Response was not an HTTPURLResponse.")
            }
    }
)
```

### [Controlling Publishing with Connectable Publishers](https://developer.apple.com/documentation/combine/controlling-publishing-with-connectable-publishers)
---

`URL Session`은 `URLSession.DataTaskPublisher`가 downstream subscriber 들이 모두 정상적으로 연결되었을 때 동작을 수행하도록 할 수 있다. `ConnectablePublisher` 프로토콜의 `makeConnectable` 메서드를 통해 `Publishers.Share`을 감싸고, subscriber들이 정상적으로 연결된 이후, `connect()` 메서드를 호출하면 된다. 

``` swift
let url = URL(string: "https://example.com/")!
let connectable = URLSession.shared
    .dataTaskPublisher(for: url)
    .map() { $0.data }
    .catch() { _ in Just(Data() )}
    .share()
    .makeConnectable()


cancellable1 = connectable
    .sink(receiveCompletion: { print("Received completion 1: \($0).") },
          receiveValue: { print("Received data 1: \($0.count) bytes.") })


DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
    self.cancellable2 = connectable
        .sink(receiveCompletion: { print("Received completion 2: \($0).") },
              receiveValue: { print("Received data 2: \($0.count) bytes.") })
}


DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    self.connection = connectable.connect()
}
```