# Chatper2.피드백활용

시스템을 변경하는 방법은 크게 두 가지로 나눌 수 있다.
- 편집 후 기도하기
- 보호 후 수정하기

### '편집 후 기도하기'
- 코드 변경 계획을 신중하게 세우고, 변경 대상 코드를 이해했는지 확인한 후 비로소 변경 작업에 들어간다.
- 변경 완료 후에는 시스템을 실행해서 변경 사항이 제대로 동작하고 무언가 손상된 동작이 없는지 자세히 조사한다. 
- 겉보기에는 신중하고 매우 전문적인 방식처럼 보이지만 안전성이 높아진다는 보장은 없다. 

### '보호 후 수정하기' 
- 기본 개념은 소프트웨어를 변경할 때 '안전망'을 이용하자는 것이다.
- 작업 대상 코드 위에 망토를 덮어놓음으로써 변경에 따른 문제가 발생해도 나머지 코드에 미치는 영향을 최소화하는 것을 의미한다.
- 망토로 덮는다는 것은 테스트 루틴으로 코드를 덮는다는 것과 같다. </br>
- '보호 후 수정하기' 방식도 신중한 작업을 필요로 하지만, 테스트 루틴으로부터 피드백을 받을 수 있기 때문에 더욱 신중하게 변경 수행이 가능하다.

### 회귀 테스트
- 작업 결과의 정확성을 보여주기 위한 테스트이다. 
- 주기적으로 테스트를 실행해 정상적인 동작 여부를 확인하고 제대로 동작하는지 조사하는 것을 의미한다. 
- 매우 좋은 개념임에도 자주 사용하지 않는데, 그 이유는 조직 내에서 시간이 오래 걸려서이다. 오히려 시간을 단축할 수 있는 단위 테스트를 선호한다. 

### 단위 테스트
- 기본 개념은 독립된 개별 소프트웨어 컴포넌트를 테스트하는 것이다.
  - 컴포넌트란 시스템의 가장 원자적인 동작 단위를 의미한다.
- 절차적 프로그래밍에서 단위는 '보통 함수', 객체 지향 프로그래밍에서는 '클래스'를 의미한다.
- 함수나 클래스의 분리 테스트는 단위 테스트의 의미상 매우 중요하다. 왜?
- 소프트웨어의 오류는 결국 소프트웨어의 각 조각들이 통합될 때 발생하므로, 좀 더 넓은 기능 영역을 포괄하는 대규모 테스트가 더 중요하다고 생각한다.
- 물론 대규모 테스트도 중요하지만 몇 가지 문제점이 있다.

  1. 오류 위치 파악
  - 테스트 루틴이 테스트 대상으로부터 멀어질수록, 실패가 의미하는 바를 파악하기 힘들어진다.
  2. 실행 시간
  - 테스트 루틴의 길이가 길어질수록 실행하는 데 오랜 시간이 걸린다.
  3. 커버리지
  - 코드 조각과 코드 조각을 실행시키는 값들의 연결 관계는 파악하기 어렵다.
 
- 따라서 단위테스트는 코드 조각을 독립적으로 테스트하며 대규모 테스트의 단점을 보완할 수 있다.
- 좋은 단위 테스트는 아래와 같다.
  1. 실행 속도가 빠르다. (실행에 0.1초가 걸리는 단위 테스트는 속도가 느린 단위 테스트)
  2. 오류 위치 파악에 도움이 된다.

- 단위 테스트는 단위 테스트가 아니다.
  1. 데이터베이스와 연동한다.
  2. 네트워크를 통해 통신한다.
  3. 파일시스템을 건드린다.
  4. 테스트 실행을 위해 특별한 작업을 같이 해야 한다.
    
### 상위 수준의 테스트
- 애플리케이션 내의 시나리오나 상호작용을 테스트하는 상위 수준의 테스트 역시 필요할 때가 있다.
- 상위 수준의 테스트를 통해 다수 클래스의 동작을 한 번에 확인할 수 있다.

### 테스트를 통한 코드 보호
- 그렇다면 레거시 프로젝트에서 변경 작업을 할 때 어떤 일부터 시작해야 할까?
- 가장 먼저 변경을 가할 코드 주위에 테스트 루틴을 배치하는 것이다.

- 아래의 사진으로 예시를 들어보자.
<img width="505" alt="스크린샷 2024-06-05 오후 5 17 56" src="https://github.com/seungchan2/EffectivelyLegacyCode/assets/80672561/32ab8286-c98d-4bba-9e92-d95c1e38b10e">


### 수정 전 코드
```swift
class InvoiceUpdateResponder {
    private let dbConnection: DBConnection
    private let servlet: InvoiceUpdateServlet

    init(dbConnection: DBConnection, servlet: InvoiceUpdateServlet) {
        self.dbConnection = dbConnection
        self.servlet = servlet
    }

    func getResponseText() -> String {
        return ""
    }
}

class Invoice {
    init() {
        // Invoice 생성자
    }

    func getValue() -> Int {
        // getValue 메서드
        return 0
    }
}
```

### 수정 후 코드
```swift
// InvoiceUpdateServlet 인터페이스
protocol InvoiceUpdateServlet {
    func execute(request: HttpServletRequest, response: HttpServletResponse)
    func buildUpdate()
}

// DBConnection 인터페이스
protocol DBConnection {
    func getInvoices(criteria: Criteria) -> [Invoice]
}

// InvoiceUpdateResponder 클래스
class InvoiceUpdateResponder {
    private let dbConnection: DBConnection
    private let invoiceIDs: [String]

    init(dbConnection: DBConnection, invoiceIDs: [String]) {
        self.dbConnection = dbConnection
        self.invoiceIDs = invoiceIDs
    }

    // 변경된 메서드
    func getResponseText() -> String {
        // InvoiceUpdateServlet로부터 전달받은 invoiceIDs를 사용하여 응답 텍스트 생성
        return ""
    }

    func update() {
        // 업데이트 로직
    }
}

// DBConnection 인터페이스 구현
class RealDBConnection: DBConnection {
    func getInvoices(criteria: Criteria) -> [Invoice] {
        // 실제 데이터베이스에서 Invoice를 가져오는 로직
        return []
    }
}

// 기존 클래스나 인터페이스
class Criteria {
    // 기준에 대한 정보를 포함하는 클래스
}
```
- #### 문제점
- `InvoiceUpdateResponder`는 DBConnection과 `InvoiceUpdateServlet`을 필요로 함.
- 데이터베이스 설정 및 사용은 비효율적이며 테스트를 느리게 함.
- `InvoiceUpdateServlet`의 생성이 복잡하며 필요한 상태로 만들기 어려움.

#### 해결책
- `InvoiceUpdateResponder`는 `InvoiceUpdateServlet`에서 필요한 정보만 전달받도록 변경.
- DBConnection의 의존성을 인터페이스(IDBConnection)로 대체.

#### 결론
- 의존성이 높은 클래스는 테스트가 어렵기 때문에 이를 해결하기 위해 인터페이스 도입 및 필요한 데이터만 전달하는 방식으로 변경.
- 레거시 작업 코드 작업은 코드를 좀 더 쉽게 변경할 수 있도록 의존 관계를 제거하는 작업을 포함한다.

#### 수정 후 사진
<img width="505" alt="스크린샷 2024-06-07 오후 1 38 05" src="https://github.com/seungchan2/EffectivelyLegacyCode/assets/80672561/c9d10ea8-b071-4b96-8aa0-db16ca7eadd5">

### 레거시 코드를 변경하는 순서
1. 변경 지점을 식별한다.
2. 테스트 루틴을 작성할 위치를 찾는다.
3. 의존 관계를 제거한다.
4. 테스트 루틴을 작성한다.
5. 변경 및 리팩토링을 수행한다.
