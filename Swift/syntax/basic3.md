
## Protocol
1. 구조체, 클래스, 열거형은 프로토콜을 채택해서 특정 기능을 실행하기 위한 프로토콜의 요구사항을 실제로 구현할 수 있다.
2. 프로토콜은 정의를 하고 제시를 할 뿐 스스로 기능을 구현하지는 않는다. (조건만 정의)
3. 하나의 타입으로 사용되기 때문에 아래와 같이 타입 사용이 허용되는 모든 곳에 프로토콜을 사용할 수 있다.
> - 함수, 메소드, 이니셜라이저의 파라미터 타입 혹은 리턴 타입
> - 상수, 변수, 프로퍼티의 타입
> - 배열, 딕셔너리의 원소타입

<br>
구조체, 클래스, 열거형에 ":"을 붙여 채택하고 ","로 구분하여 명시<br>
SubClass의 경우 SuperClass를 가장 앞에 명시

```
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
```
protocol Student {
    var height: Double { get set }
    var name: String { get }
    static var schoolNumber: Int { get set }
}
```

### 전체 예시
Coach라는 Protocol을 만들었고 Mourinho라는 구조체에 채택하여 사용하는데 따로 name, currentTeam등이 빠지면 error가 나게 된다! 
```
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

## Delegation(위임)
Delegation이란 클래스나 구조체의 인스턴스에 특정 행위에 대한 책임을 넘기는 디자인 패턴 중 하나
- Delegation된 책임을 캡슐화하는 프로토콜을 정의하는것으로 표현
- Delegation은 특정 액션에 응답하거나 해당 소스의 기본 타입을 알 필요 없이 외부 소스에서 데이터를 검색하는 데 사용할 수 있다.

### 각 필요한 프로토콜과 채택한 class, Dice 타입 
```
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
```
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
```
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
```
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
```
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

## Extension