# Chapter6.고칠 것은 많고 시간은 없고

- 실제로 코드 변경을 위해 의존 관계를 제거하고 테스트 루틴을 작성하는 것은 개발자의 시간을 많이 빼앗는다.
- 하지만 결국 개발 시간과 시행착오를 줄여준다.
- 테스트 루틴은 소프트웨어 기능상의 문제를 파악하는 일을 돕는다.

- 테스트 루틴의 사용은 개발 작업의 속도를 높이므로 대부분의 개발 팀에게 중요하다고 말할 수 있다.
- 기능 추가에 시간이 얼마나 걸릴지 모르거나 주어진 시간이 많지 않을 때, 테스트 루틴없이 기능 구현에 초점을 맞춘다.
- 하지만 충분한 시간이 있을 때, 해야지 마음을 먹고 결국 테스트와 리팩토링을 하지 않는다.

### 발아 메소드
#### 수정 전 코드
```swift
class TransactionGate {
    func postEntries(entries: [Entry]) {
        for entry in entries {
            entry.postDate()
        }
        transactionBundle.getListManager().add(entries)
    }
}
```
#### 수정 후 코드
```swift
class TransactionGate {
    func postEntries(entries: [Entry]) {
        for entry in entries {
            entry.postDate()
        }
        transactionBundle.getListManager().add(entries)
    }
}

class Entry {
    func postDate() {
        // 날짜 게시 로직
    }
}

class ListManager {
    func add(_ entries: [Entry]) {
        // 리스트에 항목 추가 로직
    }
}

class TransactionBundle {
    func getListManager() -> ListManager {
        // ListManager 인스턴스 반환 로직
    }
}
```
- 반복문이 수행되기 전에 메소드의 첫 부분에 추가하거나, 반복문 내에서 테스트 루틴을 작성할 수 있다.
- 단순한 변경처럼 보이지만, 상당히 무리가 있는 방법이다.
- 두 개의 동작이 섞여 있으며, 새로 추가한 임시 변수는 새로운 코드를 불러들이기가 쉽다.

```swift
class TransactionGate {
    var transactionBundle: TransactionBundle

    init(transactionBundle: TransactionBundle) {
        self.transactionBundle = transactionBundle
    }

    func postEntries(entries: [Entry]) {
        let entriesToAdd = uniqueEntries(entries: entries)
        for entry in entriesToAdd {
            entry.postDate()
        }
        transactionBundle.getListManager().add(entriesToAdd)
    }

    func uniqueEntries(entries: [Entry]) -> [Entry] {
        var result = [Entry]()
        for entry in entries {
            if !transactionBundle.getListManager().hasEntry(entry) {
                result.append(entry)
            }
        }
        return result
    }
}
```
- 테스트 메소드를 작성하고 호출해서 사용한다.
- 추가로 연동하는 코드가 더 필요하다면, 새로 클래스를 하나 만들고 메소드들을 해당 클래스에 작성하면 된다.

1. 어느 부분에 코드 변경이 필요한지 식별한다.
2. 메소드 내의 특정 위치에서 일련의 명령문으로서 구현할 수 있는 변경이라면, 필요한 처리를 수행하는 신규 메소드를 호출하는 코드를 작성한 후, 주석 처리한다.
3. 호출되는 메소드가 필요로 하는 지역 변수를 확인하고, 이 변수들을 신규 메소드 호출의 인수로서 전달한다.
4. 호출하는 메소드에 값을 반환해야 하는지 여부를 결정한다. 값을 반환해야 한다면, 반환 값을 변수에 대입하도록 호출 코드를 변경한다.
5. 새롭게 추가되는 메소드를 테스트 주도 개발 방법을 사용해 작성한다.
6. 앞서 주석 처리했던 신규 메소드 호출 코드의 주석을 제거한다.

- 독립된 한 개의 기능으로서 코드를 추가하는 경우나 테스트 루틴이 아직 준비되지 않은 경우 발아 메소드의 사용을 권장한다.

### 장점과 단점
#### 단점
1. 소스 메서드와 클래스의 포기
  - 발아 메서드를 사용하면, 소스 메서드와 그 클래스에 대해 일시적으로 포기하는 것과 같다.
  - 테스트를 거치지 않고 개선도 하지 않은 채 새로운 기능만을 추가하는 것이다.
  - 이는 실용적인 선택일 수 있지만, 코드가 미완성 상태로 남아있는 느낌을 준다.

2. 코드의 복잡성 증가:
  - 소스 메서드에 복잡한 코드가 많고 새로운 메서드가 추가되면, 코드가 왜 이렇게 구성되었는지 이해하기 어려울 수 있다.
  - 이는 소스 메서드가 테스트 가능한 상태가 되었을 때 추가적인 작업이 필요함을 의미한다.
  
#### 장점
1. 신규 코드와 기존 코드의 명확한 분리
  - 발아 메서드를 사용하면 새로운 코드와 기존 코드를 명확하게 분리할 수 있다.
  - 기존 코드를 즉시 테스트할 수 없더라도, 변경 사항을 별도로 볼 수 있고, 새로운 코드와 기존 코드 간의 명확한 인터페이스를 가질 수 있다.

2. 변수의 영향을 명확히 파악
  - 모든 변수가 어떻게 영향을 받는지 쉽게 파악할 수 있어 코드의 문맥에서 올바른지 판단하기가 쉬워진다.


- 발아 메서드는 일시적으로 기존 코드의 복잡성을 피하면서 새로운 기능을 추가할 수 있는 실용적인 방법이다.
- 그러나, 이는 최종적인 해결책이 아니며, 나중에 소스 메서드와 클래스를 테스트 가능한 상태로 만들고 개선하는 추가 작업이 필요하다.

### 발아 클래스
- 발아 메소느는 복잡하게 얽힌 의존 관계에서 효과적이지 않다.
- 객체 생성과 관련된 의존 관계가 많이 존재하기 때문에 클래스의 인스턴스의 생성이 어려울 수 있다.

### 포장 메소드
- 추가 코드가 기존 코드와 동시에 실행될 수 있는데, 과도하게 사용되면 품질을 저하시킨다.
- 이미 코드가 너무 복잡해져 봉합 기법을 사용하지 않으면 분리하기 힘들다.

```swift
class Employee {
   .
   .
   .

    private func dispatchPayment() {
        let amount = Money()
        for card in timecards {
            if payPeriod.contains(date) {
                amount.add(hours: card.getHours() * payRate)
            }
        }
        payDispatcher.pay(employee: self, date: date, amount: amount)
    }

    public func pay() {
        logPayment()
        dispatchPayment()
    }

    private func logPayment() {
    }
}

class Timecard {
    func getHours() -> Double {
        return 0.0
    }
}

class PayPeriod {
    func contains(_ date: Date) -> Bool {
        // 날짜 포함 여부 확인 로직
        return true
    }
}

class PayDispatcher {
    func pay(employee: Employee, date: Date, amount: Money) {
    }
}

class Money {
    private var amount: Double = 0.0

    func add(hours: Double) {
        amount += hours
    }
}

```
- pay() -> dispatchPayment()로 변경하고 private로 만들었다.
- dispatchPayment()를 호출하는 새로운 pay 메소드를 작성했다.
- 기존 메소드와 이름이 같은 메소드를 생성하고 기존 코드에 처리를 위임한다.

### 장점과 단점
#### 장점

1. 유지보수성 향상
   - 새로운 기능을 추가할 때 기존 로직에 최소한의 영향을 미치므로 코드의 유지보수가 용이하다.

2. 가독성 향상
   - pay() 메서드는 로그 기록과 지급 처리를 명확히 분리하여 각 메서드의 역할을 더 명확하게 한다.
3. 클라이언트 코드 변경 없음
   - 기존의 pay() 메서드를 호출하는 클라이언트는 변경 사항을 인지할 필요 없이 동일한 메서드를 호출할 수 있다.

#### 단점
1. 기존 로직과 새로운 기능의 독립성
   - 새로운 기능이 기존 로직과 얽혀서는 안 되며, 기존 로직 이전이나 이후에 실행되어야 한다.
2. 기존 메서드 이름 변경
   - 기존 메서드의 이름을 변경해야 할 수 있으며, 새로운 이름을 정하는 것이 어려울 수 있다.

### 포장 메소드 동작 단계
1. 변경해야 할 메소드를 식별한다.
2. 변경이 메소드 내의 특정 위치에서 구현 가능하다면, 메소드 이름을 바꾸고 기존 메소드와 동일한 이름과 시그니처를 갖는 메소드를 새로 작성한다.
3. 새로운 메소드에서 기존 메소드를 호출한다.
4. 새로운 기능을 위한 메소드를 테스트 주도 개발을 통해 작성하고, 이 메소드를 단계 2에서 작성한 신규 메소드에서 호출한다.

### 포장 클래스
```swift
protocol Employee {
    func pay()
}

class RegularEmployee: Employee {
    func pay() {
        let amount = Money()
        for card in timecards {
            if payPeriod.contains(date) {
                amount.add(hours: card.getHours() * payRate)
            }
        }
        payDispatcher.pay(employee: self, date: date, amount: amount)
    }

    // 필요한 프로퍼티 및 메서드들
    var timecards: [Timecard] = []
    var payPeriod: PayPeriod
    var date: Date
    var payRate: Double
    var payDispatcher: PayDispatcher

    init(timecards: [Timecard], payPeriod: PayPeriod, date: Date, payRate: Double, payDispatcher: PayDispatcher) {
        self.timecards = timecards
        self.payPeriod = payPeriod
        self.date = date
        self.payRate = payRate
        self.payDispatcher = payDispatcher
    }
}

class LoggingEmployee: Employee {
    private let employee: Employee

    init(employee: Employee) {
        self.employee = employee
    }

    func pay() {
        logPayment()
        employee.pay()
    }

    private func logPayment() {
        // 로그 기록 로직
    }
}

// 관련 클래스 및 구조체 정의
class Timecard {
    func getHours() -> Double {
        return 8.0 // 예시로 8시간을 반환
    }
}

class PayPeriod {
    func contains(_ date: Date) -> Bool {
        return true // 예시로 항상 true를 반환
    }
}

class PayDispatcher {
    func pay(employee: Employee, date: Date, amount: Money) {
        // 지급 로직
    }
}

class Money {
    private var amount: Double = 0.0

    func add(hours: Double) {
        amount += hours
    }
}
```
- 데코레이터 패턴이라고 불린다.
-  다른 클래스를 감싸는 클래스의 객체를 생성하여 전달하고ㅡ 감싸는 클래스는 감싸고 있는 클래스와 동일한 인터페이스를 가져야 하므로 클라이언트는 래퍼와 작업하고 있다는 것을 알 수 없다.
- 예제에서 LoggingEmployee는 Employee의 데코레이터이다.
- pay() 메서드와 클라이언트가 사용하는 Employee의 다른 메서드를 가져야 한다.

#### 장점
1. 새로운 기능을 기존 코드에 영향을 주지 않고 추가할 수 있다.
2. 클래스의 동작을 동적으로 확장할 수 있다.

#### 단점
1. 클래스가 많아지면 관리가 복잡해질 수 있다.
2. 모든 메서드를 래핑해야 하므로 코드가 중복될 수 있다.

### 포장 클래스의 사용 고려
1. 추가하려는 동작이 완전히 독립적이며, 구현에 의존적인 동작이나 관련 없는 동작으로 기존 클래스를 오염시키고 싶지 않을 경우
2. 클래스가 비대해져서 더 이상 키우고 싶지 않은 경우
