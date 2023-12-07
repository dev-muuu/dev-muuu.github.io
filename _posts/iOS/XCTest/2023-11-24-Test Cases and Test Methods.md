---
layout: post
title: Test Cases and Test Methods
categories: XCTest
tags: [iOS, XCTest]
---

## [Apple Documentation](https://developer.apple.com/documentation/xctest#1612636)

테스트 함수는 코드의 특정 부분을 테스트할 수 있는 작은 자체적인 함수입니다. 테스트 케이스는 연관된 테스트 함수들의 그룹입니다. 

## Defining Test Cases and Test Methods
---

### Overview
---

`XCTestCase`의 하위 클래스에 하나 이상의 테스트 함수들을 작성하여 관련 테스트들을 그룹화할 수 있습니다. 

프로젝트에 테스트를 추가하는 방법:
- 테스트 타켓에 대한 테스트 케이스인` XCTestCase`의 하위 클래스를 생성합니다.
- 테스트 케이스에 대해 하나 이상의 테스트 함수를 추가합니다. 
- 각각의 테스트 함수에 대해 하나 이상의 테스트 assertion을 추가합니다. 

테스트 함수는 XCTestCase 하위 클래스의 인스턴스 메서드이지만 매개변수와 반환 값이 없으며, 네이밍이 `test`로 시작되어야 합니다. 테스트 함수들은 XCTest 프레임워크에 의해 자동으로 탐지 되어집니다. 

``` swift
class TableValidationTests: XCTestCase {
    /// Tests that a new table instance has zero rows and columns.
    func testEmptyTableRowAndColumnCount() {
        let table = Table()
        XCTAssertEqual(table.rowCount, 0, "Row count was not zero.")
        XCTAssertEqual(table.columnCount, 0, "Column count was not zero.")
    }
}
```

### Asserting Test Conditions
---

코드가 예상대로 작동하는지 보장하기 위해 테스트 함수를 통해 확인할 수 있다. `XCTAssert` 함수군을 사용하여 Bool 조건, nil 또는 non-nil 값, 기대값 그리고 던져진 오류를 확인할 수 있습니다. 

## XCTestCase
---

### [Set Up and Tear Down State in Your Tests]({{ iOS/XCTest/2023-11-25-Set Up and Tear Down State in Your Tests.md | relative_url }})

## XCTest
---

