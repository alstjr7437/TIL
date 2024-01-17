# Protocol
1. 구조체, 클래스, 열거형은 프로토콜을 채택해서 특정 기능을 실행하기 위한 프로토콜의 요구사항을 실제로 구현할 수 있다.
2. 프로토콜은 정의를 하고 제시를 할 뿐 스스로 기능을 구현하지는 않는다. (조건만 정의)
3. 하나의 타입으로 사용되기 때문에 아래와 같이 타입 사용이 허용되는 모든 곳에 프로토콜을 사용할 수 있다.
> - 함수, 메소드, 이니셜라이저의 파라미터 타입 혹은 리턴 타입
> - 상수, 변수, 프로퍼티의 타입
> - 배열, 딕셔너리의 원소타입

<br>
구조체, 클래스, 열거형에 ":"을 붙여 채택하고 ","로 구분하여 명시<br>
SubClass의 경우 SuperClass를 가장 앞에 명시

```swift
protocol 프로토콜이름 {
    // 프로토콜 정의
}

// 구조체 정의
struct SomeStruct: AProtocol, AnotherProtocol {
}

// 상속받는 클래스의 프로토콜 채택
class SomeClass: SuperClass, AProtocol, AnotherProtocol {
    // 클래스 정의
}
```

### Property Requirments
프로토콜에서는 프로퍼티가 저장프로퍼티, 연산프로퍼티인지 명시하지 않고, <b>이름과 타입 그리고 gettable, settable 한지 명시</b>. (프로퍼티는 <b>항상 var로 선언.</b>)
```swift
protocol Student {
    var height: Double { get set }
    var name: String { get }
    static var schoolNumber: Int { get set }
}
```

### 전체 예시
Coach라는 Protocol을 만들었고 Mourinho라는 구조체에 채택하여 사용하는데 따로 name, currentTeam등이 빠지면 error가 나게 된다! 
```swift
protocol Coach {
    var name: String { get set }
    var currentTeam: String { get }
    func training()
    func direct()
}

struct Mourinho: Coach {
    var name: String = "Jose Mourinho"
    var currentTeam: String = "AS Roma"
    
    func training() {
        print("Traing Player")
    }
    
    func direct() {
        print("Direct Game")
    }
}

let mourinho = Mourinho()
print("\(mourinho.name), \( mourinho.currentTeam)")
mourinho.training()
mourinho.direct()

//    Jose Mourinho, AS Roma
//    Traing Player
//    Direct Game
```

# Delegation(위임)
Delegation이란 클래스나 구조체의 인스턴스에 특정 행위에 대한 책임을 넘기는 디자인 패턴 중 하나
- Delegation된 책임을 캡슐화하는 프로토콜을 정의하는것으로 표현
- Delegation은 특정 액션에 응답하거나 해당 소스의 기본 타입을 알 필요 없이 외부 소스에서 데이터를 검색하는 데 사용할 수 있다.

### 각 필요한 프로토콜과 채택한 class, Dice 타입 
```swift
protocol RandomNumberGenerator {
    func random() -> Double
}

class LinearCongruentialGenerator: RandomNumberGenerator {
    var lastRandom = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0
    func random() -> Double {
        lastRandom = ((lastRandom * a + c).truncatingRemainder(dividingBy:m))
        return lastRandom / m
    }
}

class Dice {
    let sides: Int
    let generator: RandomNumberGenerator
    init(sides: Int, generator: RandomNumberGenerator) {
        self.sides = sides
        self.generator = generator
    }
    func roll() -> Int {
        return Int(generator.random() * Double(sides)) + 1
    }
}
```

### Delegation 코드
```swift
protocol DiceGame {
    var dice: Dice { get }
    func play()
}

/// DiceGame의 책임을 DiceGameDelegate를 따르는 인스턴스에 위임
protocol DiceGameDelegate: AnyObject { // AnyObject로 선언하면 클래스만 이 프로토콜을 채택할 수 있는 속성이 된다.
    func gameDidStart(_ game: DiceGame)
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int)
    func gameDidEnd(_ game: DiceGame)
}
```

### DiceGame 프로토콜을 채택하며, DiceGameDelegate에게 진행상황을 알리는 코드이다.
```swift
class SnakesAndLadders: DiceGame { // 프로토콜의 조건에 맞추기위해 dice라는 프로퍼티는 gettable하게 구현되어 있고, play()메소드가 구현되어 있다.
    let finalSquare = 25
    let dice = Dice(sides: 6, generator: LinearCongruentialGenerator())
    var square = 0
    var board: [Int]
    init() {
        board = Array(repeating: 0, count: finalSquare + 1)
        board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
        board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
    }
    // 강한 참조 순환을 막기위해 약한 참조를 함
    weak var delegate: DiceGameDelegate? // 게임 진행에 반드시 필요한 것은 아니기 때문에 옵셔널로 정의
    /// 게임의 전체 로직이 들어있는 메소드
    func play() { // 바로 위에 정의한 delegate가 DiceGameDelegation 옵셔널타입이므로 play() 메소드는 delegate의 메소드를 호출할때마다 옵셔널 체이닝을 한다.
        square = 0
        /// DiceGameDelegation의 게임 진행상황을 tracking하는 메소드(게임 시작)
        delegate?.gameDidStart(self)
        gameLoop: while square != finalSquare {
            let diceRoll = dice.roll()
            /// DiceGameDelegation의 게임 진행상황을 tracking하는 메소드(게임 진행)
            delegate?.game(self, didStartNewTurnWithDiceRoll: diceRoll)
            switch square + diceRoll {
            case finalSquare:
                break gameLoop
            case let newSquare where newSquare > finalSquare:
                continue gameLoop
            default:
                square += diceRoll
                square += board[square]
            }
        }
        /// DiceGameDelegation의 게임 진행상황을 tracking하는 메소드(게임 종료)
        delegate?.gameDidEnd(self)
    }
}
```
### DiceGameDelegate 프로토콜을 채택한 DiceGameTracker클래스에 대한 코드이다.
```swift
class DiceGameTracker: DiceGameDelegate { // 프로토콜의 조건에 맞추기위해 3개의 메소드가 구현되어 있다.
    var numberOfTurns = 0
    func gameDidStart(_ game: DiceGame) {
        numberOfTurns = 0
        if game is SnakesAndLadders { // SnakesAndLadders 클래스의 인스턴스가 매개변수로 들어오면 실행
            print("Started a new game of Snakes and Ladders")
        }
        print("The game is using a \(game.dice.sides)-sided dice")
    }
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int) {
        numberOfTurns += 1
        print("Rolled a \(diceRoll)")
    }
    func gameDidEnd(_ game: DiceGame) {
        print("The game lasted for \(numberOfTurns) turns")
    }
}
```

### 실행 결과 
```swift
let tracker = DiceGameTracker()
let game = SnakesAndLadders()
game.delegate = tracker
game.play()
// Started a new game of Snakes and Ladders
// The game is using a 6-sided dice
// Rolled a 3
// Rolled a 5
// Rolled a 4
// Rolled a 5
// The game lasted for 4 turns
```

# Extension
타입에 기능을 추가하기 위해 사용되는 문법<br>
새로운 프로토콜을 채택하기 위해 Extension을 사용할 수도 있음.
- 상속 역시 기존 타입에서 확장할 수 있지만, 새로운 클래스를 만들면서 기능을 확장한다는 차이점이 있음.
- Extension은 그냥 기능을 덛붙힌다고 보면 됨.(수평적 확장)
- 기능들을 모음으로써 가독성을 높힐 수 있음.
- 프로퍼티는 연산 프로퍼티만 사용할 수 있음.
```swift
extension SomeType {
    // 구현부
}

// 새로운 프로토콜 채택
extension SomeType: SomeProtocol {
    // 구현부
}
```

### 예시
```swift
extension String {
    var length: Int {
        var string: NSString = NSString(string: self)
        return sting.length
    }
}
```

### Extension에 조건 추가
where를 사용하여 특정 조건을 만족시킬때만 기능 확장하거나 프로토콜 채택하도록 제한 가능
```swift
extension Array: SomeProtocol where Element: Int {
    // 구현부
}
```

### Extension을 사용한 프로토콜 채택 선언
타입이 이미 프로토콜의 모든 요구사항을 만족하지만 아직 해당 프로토콜을 채택한다고 명시하지 않은 경우 형식에 빈 Extension을 사용해서 프로토콜을 채택하도록 할 수 있다.
```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}
extension Hamster: TextRepresentable {}
```

## Protocol 상속
클래스 상속과 같이 프로토콜도 상속할 수 있음. 마찬가지로 ","로 구분
```swift
protocol Movable {
    func go(to destination: String)
}

protocol OnBoardable {
    var numberOfPassangers: Int { get }
}

protocol Vehicle: Movable, OnBoardable { }
// typealias Vehicle = Movable & OnBoardable  <- 위와 같은 효과

struct Car: Vehicle {
    func go(to destination: String) {
        print("\(destination)(으)로 갑니다")
    }
    
    var numberOfPassangers: Int = 4
}

var car = Car(numberOfPassangers: 9)
car.go(to: "집") 
// 집(으)로 갑니다
print(car.numberOfPassangers) 
// 9
```

## 프로토콜 합성
동시에 여러 프로토콜을 따르는 타입을 선언할 수 있음.
```swift
protocol Named {
  var name: String { get }
}
protocol Aged {
  var age: Int { get }
}
struct Person: Named, Aged {
  var name: String = "Aiden"
  var age: Int = 27
}
```

## 프로토콜 타입 확인
일반적인 타입 확인과 마찬가지로 is,as를 사용
- is : 앞에있는 타입이 뒤에있는 프로토콜을 채택하고 있는지 확인 (반환타입 Bool)
- as? : 앞에있는 타입이 뒤에있는 프로토콜을 채택하고 있는 경우 해당 타입을 프로토콜 타입으로 다운케스트, 그렇지 않은 경우는 nil 반환
- as! : 앞에있는 타입을 뒤에있는 프로토콜 타입으로 다운캐스트 실패시 런타임 에러 발생

# Optional Protocol Requirements
프로토콜을 선언하면서 필수 구현이 아닌 선택적 구현 조건을 정의할 수 있음.

## @objc를 사용한 방법
- Objective-C를 사용한 방법
- @objc 키워드를 프로토콜 앞에 붙이고 메소드나 프로퍼티에는 @objc와 optional을 붙임
- @objc 프로토콜은 클래스만 채택 가능
```swift
@objc protocol Person {
  @objc optional var name: String {get}
  @objc optional func speak()
}

class Aiden: Person {
  func notChoice() {
    print("name, speak()를 사용하지 않았습니다.")
  }
}

let aiden = Aiden()
aiden.notChoice() // name, speak()를 사용하지 않았습니다.

// Person을 채택하면서 구현은 하나도 하지 않는 클래스를 선언할 수는 있지만 권장하지 않는다.
```
### @objc의 단점 : 사용자가 정의한 타입을 사용할 수 없다.
```swift
//Property cannot be marked @objc 에러 발생
struct CustomType {
  var property: String = "특수타입"
}
@objc protocol Person {
  @objc optional var name: CustomType {get}
  @objc optional func speak()
}
```
### 해결 -> NSObject를 상속받아서 -> 클래스만 가능

```swift
// NSObject를 상속받아야 해서 클래스만 가능
class CustomType: NSObject {
    var property: String = "특수타입"
}

@objc protocol Person {
    @objc optional var name: CustomType {get}
    @objc optional func speak()
}

class Aiden: Person {
    func notChoice() {
        print("name, speak()를 사용하지 않았습니다.")
    }
}

let aiden = Aiden()
aiden.notChoice()
```

## Extension을 활용한 방법
@objc의 단점을 해결하는 또다른 방법으로, 프로토콜을 정의한 후 구현하고 싶은 기능만 확장해서 이를 채택하게 하는 방법이있다.
- 해당 프로토콜을 채택하는 타입들이 기능들을 따로 구현하지 않고, 기능이 구현된 상태로 그대로 받길 원할 때도 사용
```swift
struct CustomType {
    var name: String
}

protocol Person {
    var specialPeople: CustomType {get}
    func speak()
    func sleep()
}

extension Person {
    func speak() {
        print(specialPeople.name+"이 말합니다.")
    }
}

class Aiden: Person { // name과 sleep() 메소드만 구현하면 된다.
    var specialPeople: CustomType = CustomType(property: "Aiden")
    func sleep() {
        print(specialPeople.name+"잡니다.zzz")
    }
}

let aiden = Aiden()
aiden.speak() // Aiden이 말합니다.
aiden.sleep() // Aiden잡니다.zzz
```
- 이 방법을 사용하면 사용자정의 타입을 사용할 수 있다.
- 이미 Person의 speak() 메소드가 구현되었으므로 Person을 채택하는 Aiden 클래스는 specialPeaple과 sleep()만 구현하면 된다.
- speak()메소드가 기능이 없게 놔두고 싶다면 Extension을 사용하고 메소드를 빈 구현체로 구현하면된다.

## @objc, Extension 비교
@objc를 사용한 방법과 Extension을 활용한 방법은 각각의 장단점이 있는것 같다. <br>
Extension을 활용한 방법에서 염려되었던 부분은
1. Extension하고 메소드를 빈 구현체로 구현하면 기능은 없지만 그 메소드는 남아있다는 점.
2. return타입을 가지는 경우 반드시 return값을 정해줘야 하기에 메모리 낭비가 생길 수 있다는 점.

@objc를 사용한 방법에서 염려되었던 부분은
1. class만 채택할 수 있다는 점.
2. 사용자정의 타입도 클래스로만 정의 가능하다는 점.

만약 프로토콜을 채택하는 타입이 클래스고 사용자 정의 타입이 없거나 클래스로 정의할 타입이라면 @objc <br>
그 외에는 Extension을 활용한 방법이 좋은 것 같다.