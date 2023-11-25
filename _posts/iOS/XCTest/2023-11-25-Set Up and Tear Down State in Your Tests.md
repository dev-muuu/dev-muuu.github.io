---
layout: post
title: Set Up and Tear Down State in Your Tests
categories: XCTest
tags: [iOS, XCTest]
---

## [Apple Documentation](https://developer.apple.com/documentation/xctest/xctestcase/set_up_and_tear_down_state_in_your_tests)

테스트를 실행하기 전 초기 상태를 준비하고, 테스트가 완료된 이후 자원을 청소합니다.

## Overview
---

코드에서 올바른 결과가 나오는지 일관적이고 확실하게 확인하려면 테스트는 예측 가능한 알려진 상태에서 시작해야 합니다. 테스트 클래스의 모든 테스트 방법에 대해 한 번씩 상태를 설정해야 하는 경우도 있고, 각 테스트 방법 전에 알려진 상태로 리셋해야 하는 경우도 있습니다.

각 테스트 방법이나 테스트 클래스가 완료된 후에는 임시 파일이나 스크린샷과 같이 필요없는 파일을 삭제할 수도 있습니다. 또는 고장 진단에 도움이 되도록 테스트 후 최종 상태를 캡처할 수도 있습니다.

테스트를 위해 알려진 상태를 준비하고, 테스트 후 임시 파일을 XCTest 및 XCTestCase의 방법을 사용하여 정리합니다.

## Decide When to Set Up and Tear Down State in Your Test Class
---

테스트 케이스를 실행할 때, XCTest는 처음에 XCTestCase의 setUp() 클래스 메서드를 호출합니다. 이 메서드를 사용하여 테스트 클래스의 모든 테스트 메서드에 공통된 상태를 설정합니다. 

XCTest가 각각의 테스트 메스드를 실행할 때, 설정과 해체 메서드를 아래의 순서로 호출합니다.

1. XCTest는 각각의 테스트 메서드가 실행될 때 설정 메서드를 실행합니다. `setUp() async throws` > `setUpWithError()` > `setUp()`의 순서로 실행됩니다. 
2. XCTest가 테스트 메서드를 실행합니다. 
3. XCTest가 테스트 메서드에 추가한 teardown 블럭을 LIFO(last in, first out) 순서로 실행합니다. 테스트 메서드 실행 이후 이 블럭을 통해 상태를 해제하고 자원을 해제합니다.
4. XCTest는 각각의 테스트 메서드가 완료된 이후 해제 메서드를 한 번 실행합니다. `tearDown() async throws` > `tearDownWithError()` > `tearDown()`의 순서로 실행됩니다. 

XCTest가 모든 테스트 방법을 실행하고 테스트 클래스가 완료되면 XCTestCase tearDown() 클래스 방법을 호출합니다. 이 방법을 사용하여 테스트 클래스의 모든 테스트 방법에 공통적인 상태를 제거합니다.

## Prepare and Tear Down State for a Test Class
---

- `tearDown()`: 테스트 클래스가 완료된 후 임시 파일을 정리하거나 분석할 데이터를 캡처해야 하는 경우 사용합니다.  


## Prepare and Tear Down State for Each Test Method
---

설정 요구 조건에 가장 적합한 하나의 setUp 메서드를 선택 후, override 하여 사용합니다.

- `setUp(completion:)`: 비동기적으로 상태를 준비해야 하는 경우 사용합니다.
- `setUpWithError()`: 모든 상태를 동기화하고, 에러를 던지는 경우 사용합니다. 던져진 에러를 잡을 수 있으며, 테스트의 실패를 기록할 수 있습니다. 
- `setUp()`: 상태를 동기화하고, 에러를 다룰 필요가 없는 경우 사용합니다. 

`XCTest`는 각각의 테스트가 완료된 이후, `tearDown()` 메서드를 실행하므로 XCTest는 테스트는 `tearDown()` 메서드가 실행되는 것을 보장하지 않습니다. 만약 테스트 완료 전 크래시가 발생한다면, XCTest는 tearDown 블럭을 호출하지 않습니다. 

## Tear Down State After a Specific Test Method
---

특정 테스트 메서드를 완료한 후 즉시 teardown을 완료해야 하는 경우, 테스트 메서드에 teardown 블럭을 추가할 수 있습니다. 


``` swift 
func testMethod1() throws {
    // This is the first test method.
    // Your testing code goes here.
    addTeardownBlock {
        // XCTest executes this when testMethod1() ends.
    }
}


func testMethod2() throws {
    // This is the second test method.
    // Your testing code goes here.
    addTeardownBlock {
        // XCTest executes this last when testMethod2() ends.
    }
    addTeardownBlock {
        // XCTest executes this first when testMethod2() ends.
    }
}
```

teardown 블럭은 await을 사용하여 비동기적으로 호출될 수 있으며, 테스트가 실패했을 경우 에러를 던질 수도 있습니다. 