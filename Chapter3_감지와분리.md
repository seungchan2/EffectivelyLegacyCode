# Chapter3.감지와 분리

- 클래스들 간에 의존 관계(객체 -> 객체 -> 객체) 때문에 특정 객체들만 테스트 루틴으로 보호하는 것은 어렵다.
- 테스트 루틴을 작성하려면 테스트 대상 클래스가 다른 클래스에 주는 영향을 알아야 할 때가 있다.
- 테스트 루틴을 배치할 때 의존 관계를 제거하는 이유는 두 가지가 있다.

### 감지
- 코드 내에서 계산된 값에 접근할 수 없을 때, 이를 감지하기 위해 의존 관계를 제거한다.

### 분리
- 코드를 테스트 하네스 내에 넣어서 실행할 수 없을 때, 코드를 분리하기 위해 의존 관계를 제거한다.

```swift
class NetworkBridge {
    init(endpoints: [EndPoint]) {
        // 초기화 코드
    }
    
    func formRouting(sourceID: String, destID: String) {
        // 라우팅 형성 코드
    }
}
```
- 해당 코드는 테스트 관점에서 몇 가지 문제점이 있다.
- `NetworkBridge`클래스의 테스트 루틴을 작성하려면 실제 하드웨어를 자주 호출할 것이다.
- 클래스의 인스턴스를 생성하기 위해 실제 하드웨어의 동작은 필요없다.
- 다양한 해결 방법이 있지만 엄청난 코스트가 든다.
- 클래스의 메소드가 호출될 때의 영향을 감지할 수도 없고, 애플리케이션의 나머지 부분과 분리해서 실행할 수 없기 때문에 감지와 분리의 예시 중 하나이다.

### 협업 클래스 위장하기
- 레거시 코드를 다룰 떄의 가장 큰 문제 중 하나는 '의존 관계'이다.
- 다른 코드를 별도의 코드로 대체할 수 있다면, 변경 대상을 테스트하는 루틴을 작성할 수 있으며, 이것을 가짜 객체 혹은 위장 객체라 한다.

### 가짜 객체
- Sale 객체에 인수로서 전달되는 디스플레이 객체는 Display 인터페이스를 구현하는 클래스이기만 하면 어떤 클래스의 객체도 가능하다.
```swift
protocol Display {
    func showLine(_ line: String)
}

class Sale {
    private var display: Display
    
    init(display: Display) {
        self.display = display
    }
    
    func scan(_ barcode: String) {
        // 아이템 정보를 얻는 코드
        let itemName = item.name
        let itemPrice = "\(item.price.asDisplayText())"
        let itemLine = "\(itemName) \(itemPrice)"
        display.showLine(itemLine)
    }
} 
```
