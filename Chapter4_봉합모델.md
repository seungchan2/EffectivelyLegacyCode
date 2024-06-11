# Chapter4.봉합 모델

### 봉합
- 단위 테스트를 위해 개별 클래스를 추출하려고 하면 대부분의 경우 의존 관계를 제거해야 한다.
- 테스트를 위해 클래스 추출하는 과정을 거치다보면 좋은 설계에 대한 생각이 달라지게 된다.
- 그럼 프로그램을 어떤 관점에서 바라봐야 할까?
``` swift
class AsyncSslRec {
    private var sslInitialized = false
    private var sslRefCount = 0
    private var failureSent = false
    private var sslDll1: Library? = nil
    private var sslDll2: Library? = nil

    func initSsl() -> Bool {
        if sslInitialized { return true }
        
        unlockMutex()
        sslRefCount += 1
        sslInitialized = true
        
        freeLibrary(&sslDll1)
        freeLibrary(&sslDll2)
        
        if !failureSent {
            failureSent = true
            postReceiveError(type: .socketCallback, errorCode: .sslFailure)
        }
        
        createLibrary(&sslDll1, name: "syncesel1.dll")
        createLibrary(&sslDll2, name: "syncesel2.dll")
        
        sslDll1?.initLibrary()
        sslDll2?.initLibrary()
        
        return true
    }
    
    func postReceiveError(type: CallbackType, errorCode: ErrorCode) {
        Global.postReceiveError(type: type, errorCode: errorCode)
    }
}
```

- 해당 코드에서 `func postReceiveError()` 빼고 실행하려고 한다.
- 실제 운영 시 함수 호출이 가능하며 테스트 시에만 호출을 막아야 한다.

### 봉합
- 봉합 지점은 코드를 직접 편집하지 않고도 프로그램의 동작을 변경할 수 있는 위치를 말한다.

- postReceiveError 메서드가 테스트 시에는 호출되지 않도록 하려면, 서브클래싱을 통해 해당 메서드를 override 하는 방법을 사용할 수 있다.
```swift
class TestingAsyncSslRec: AsyncSslRec {
    override func postReceiveError(type: CallbackType, errorCode: ErrorCode) {
        // Do nothing for testing
    }
}
// 유닛 테스트 예시
func testInitSsl() {
    let sslRec = TestingAsyncSslRec()
    
    // sslRec.initSsl() 메서드를 호출하면, postReceiveError가 호출되지 않음
    let result = sslRec.initSsl()
    
    assert(result == true)
    assert(sslRec.sslInitialized == true)
    // 추가적인 검증 코드
}
```
- 이와 같은 봉합을 '객체 봉합'이라고 부른다. 호출하는 코드를 변경하지 않고, 호출되는 메소드만 변경할 수 있다.
- 그렇다면 왜 봉합을 사용해야 할까? 어떤 쓸모가 있는 것일까? -> 의존 관계 제거

### 전처리 봉합
- C와 C++에서는 컴파일 전에 매크로 전처리기가 실행되어 코드의 특정 부분을 수정할 수 있다.
- 전처리기를 이용해 디버깅이나 다양한 플랫폼 지원을 위해 조건부 컴파일을 사용할 수 있다.

- db_update 함수가 데이터베이스와 직접 통신하기 때문에 테스트 시 다른 구현으로 대체하기 어려운 상황을 해결하기 위해 전처리기를 사용한다.
- localdefs.h 파일을 통해 db_update 호출을 매크로로 대체하여 테스트 시 실제 함수 호출 대신 테스트용 코드를 실행한다.
```swift
func accountUpdate(accountNo: Int, record: DHLSRecord, activated: Bool, updateFunc: (Int, DFHLItem) -> Void = updateDatabase) {
    if activated {
        if record.dateStamped && record.quantity > MAX_ITEMS {
            updateFunc(accountNo, record.item)
        } else {
            updateFunc(accountNo, record.backupItem)
        }
    }
    updateFunc(MASTER_ACCOUNT, record.item)
}

func testAccountUpdate() {
    var lastItem: DFHLItem? = nil
    var lastAccountNo: Int? = nil
    
    // 테스트용 대체 함수 정의
    let mockUpdateFunc: (Int, DFHLItem) -> Void = { accountNo, item in
        lastItem = item
        lastAccountNo = accountNo
    }
    
    let record = DHLSRecord(dateStamped: true, quantity: 100, item: DFHLItem(), backupItem: DFHLItem())
    
    // 실제 `updateDatabase` 함수 대신 테스트용 함수 전달
    accountUpdate(accountNo: 123, record: record, activated: true, updateFunc: mockUpdateFunc)
    
    // 테스트 검증
    assert(lastItem != nil)
    assert(lastAccountNo == 123)
}
```
- 활성화 지점
  - 모든 봉합은 활성화 지점을 갖는다. 활성화 지점에서는 어느 동작을 사용할지 선택할 수 있다.
 
### 링크 봉합
- 대부분의 언어 시스템에서는 컴파일이 빌드 프로세스의 마지막 단계가 아니다.
- 컴파일러는 코드의 중간 표현을 생성하고, 이 표현은 다른 파일의 코드 호출을 포함한다.
- 링크는 이러한 표현을 결합하여 런타임에 완전한 프로그램을 만든다.
- 링크 seam은 프로그램의 동작을 코드 수정 없이 변경할 수 있는 지점을 제공하며, 빌드 스크립트나 배포 스크립트 외부에서 사용된다.
- 테스트와 실제 환경의 차이가 명확하도록 하는 것이 중요하다.

### 객체 봉합
- 객체 봉합은 객체 지향 프로그래밍 언어에서 가장 유용한 봉합 유형 중 하나이다.

```swift
cell.recalculate()
```
- cell 객체가 어떤 클래스의 인스턴스인지에 따라 호출되는 메서드가 달라진다.
```swift
class CustomSpreadsheet: Spreadsheet {
    func buildMartSheet() {
        let cell = FormulaCell(spreadsheet: self, address: "A1", formula: "=A2+A3")
        cell.recalculate()
    }
}
```
- FormulaCell이 고정되어 있기 때문에 객체 봉합이 아니다.

```swift
class CustomSpreadsheet: Spreadsheet {
    func buildMartSheet(cell: Cell) {
        cell.recalculate()
    }
}
```
- cell.recalculate()는 봉합 지점이다.
- 호출하는 쪽의 코드를 변경하지 않으면서 cell.recalculate() 메소드의 동작을 변경할 수 있다.

```swift
class CustomSpreadsheet: Spreadsheet {
    func buildMartSheet(cell: Cell) {
        Recalculate(cell)
    }
    static func Recalculate(cell: Cell) {
        ...
    }
}
```
- 정적 메서드를 비정적으로 만들어 서브클래싱하여 테스트 중에 재정의할 수 있다.
