# 기본 문법2

## 함수

func 키워드 사용<br>
() 안에 매개변수<br>
반환값이 있을 경우 -> 이용해 반환 타입 선언하기<br>
변수나 상수에 할당 가능<br>

```
// 매개변수 주는 함수
// 함수 주는 dan이라는 매개변수로 구구단 표현
func printGugu(dan: Int) {
    for i in 1...9 {
        print("\(dan) * \(i) = \(dan * i)")
    }
}
printGugu(dan: 3)
// 3 * 1 = 3 ~~

// 반환값 있는 함수
// 함수를 호출시 그 변수에 다이스 랜덤값 저장
func rollDice() -> Int {
    return Int.random(in: 1...6)
}
let random = rollDice()
print(random)
// 3
```

## 클로저
클로저 : 이름이 존재하지 않는 함수<br>
in 안에 함수 실행 내용 작성<br>
배열, 딕셔너리 같은 컬렉션 타입과 함께, filter, map, reduce 메소드 사용시 자주 활용
```
// 함수
func call(name:String){
    print("hello, \(name)")
}
let callName = call
callName("Kim")
// hello, Kim

// 클로저
let helloName = {(name: String) in print("hello, \(name)")}
helloName("Minseok")
// hello, Minseok
```

클로저 활용 예시
```
// filter
let members = ["Jason", "Greg", "Tiffany"]
let nameHasT = members.filter { name in
    return name.hasPrefix("T")
}
// ["Tiffany"]


// map
let prices = [1000, 2000, 3000]
let doubledPrices = prices.map { price in
    return price * 2
}
// [2000, 4000, 6000]

// reduce
let revenues = [100, 200, 300]
let totalRevenue = revenues.reduce(0) { partialResult, next in
    return partialResult + next
}
// 600
```

## 구조체
구조체는 원하는 데이터타입 만들때 사용<br>
struct 사용
```
// 구조체
struct Album {
    // 멤버 변수들
    // stored property
    let title: String
    let artist: String
    var isReleased = false
    
    func description() -> String {
        return "\(title) by \(artist)"
    }
    
    // 구조체 내부 멤버 변수의 값을 변경하는 경우, mutating 키워드 이용
    mutating func release() {
        self.isReleased = true
    }
}

var easyOnMe = Album(title: "Easy On Me", artist: "Adele")
print(easyOnMe.description())
// Easy On Me by Adele

print(easyOnMe.isReleased)
// false
easyOnMe.release()
print(easyOnMe.isReleased)
// true
```

## 클래스
clss 이용해 선언
### 구조체와 비슷하지만 차이점
1. 상속이 가능함(구조체 불가능)
2. 클래스는 참조(reference), 구조체는 복사(copy)함.
3. 클래스는 멤버와이즈 이니셜라이저(생성자)를 기본으로 안만들어줌.
> <b>생성자</b> : 클래스 또는 구조체 생성할때 사용하는 특별한 함수(init 선언)

```
class Employee {
    var name: String
    var hours: Int
    
    init(name: String, hours: Int) {
        self.name = name
        self.hours = hours
    }
    
    func work() {
        print("I'm working now...")
    }
    
    func summary() {
        print("I work \(self.hours) hours a day. ")
    }
}

class iOSDeveloper: Employee {
    override func work() {
        print("I'm developing iOS app now.")
    }
    
    override func summary() {
        print("I work \(self.hours/2) hours a day.")
    }
}

struct Phone {
    var modelName: String
    var manufacturer: String
    var version: Double = 1.0
}

let normalWorker = Employee(name: "Kim", hours: 8)
normalWorker.work()
normalWorker.summary()
//    I'm working now...
//    I work 8 hours a day.

let developer = iOSDeveloper(name: "Min", hours: 8)
developer.work()
developer.summary()
//    I'm developing iOS app now.
//    I work 4 hours a day.

// Reference(참조) vs. Copy(복사)
var iPhone1 = Phone(modelName: "iPhone 13", manufacturer: "Apple")
var iPhone2 = iPhone1
iPhone1.modelName = "iPhone 14"
print(iPhone1.modelName)
print(iPhone2.modelName)
//    iPhone 13
//    iPhone 14

var jrDeveloper1 = iOSDeveloper(name: "Kim", hours: 8)
var jrDeveloper2 = jrDeveloper1
jrDeveloper1.name = "Min"
print(jrDeveloper1.name)
print(jrDeveloper2.name)
//    "Min"
//    "Min"
```

## Property(프로퍼티)
class, struct, 열거형에서 소속된 변수 및 속성을 불러오는 개념
1. Stored Property(저장 프로퍼티)
    - 인스턴트의 변수나 상수 지칭
    - 사용 시점에 따라 Lazy Stored Property(지연 저장 프로퍼티)도 있음
2. Computed Property(연산 프로퍼티)
    - 직접적인 값 지정 X 연산한 결과값
3. Type Property(타입 프로퍼티)
    - 특정 타입에 사용되는 프로퍼티(클래스 변수 등)

### Stored Property(저장 프로퍼티)
1. 객체의 값(속성)을 저장하고 있는 기본적인 프로퍼티
2. 객체가 생성이 되면 자동적으로 초기화
3. 열거형(Enum)에는 지원 X
```
class FixedLengthRange {
   var firstValue: Int
   let length: Int

   init(firstValue : Int, length:Int) {
        self.firstValue = firstValue
        self.length = length
   }
}

let rangeOfThreeItems = FixedLengthRange(firstValue: 0, length: 3)

// 클래스는 기본적으로 Reference Type 데이터이므로 let으로 선언하여도 원본에 바로 접근rangeOfThreeItems.firstValue = 3
rangeOfThreeItems.length = 10 // error!
```

### Lazy stored Property(지연 저장 프로퍼티)
1. 변수가 사용된 이후에 저장되지 않는 프로퍼티
2. 값이 사용되기 전까지는 계산되지 않음.
3. lazy 키워드 사용
4. let 상수는 불가
5. lazy 프로퍼티가 초기화 되지 않은 상태에서 여러 쓰레드가 동시에 이 lazy프로퍼티에 액세스 한다면, 이 프로퍼티가 단 한번만 초기화 된다는 것을 보장할 수 없음
```
class DataImporter {
     /*
         DataImporter는 외부 파일에서 데이터를 가져오는 클래스입니다.
         이 클래스는 초기화 하는데 매우 많은 시간이 소요된다고 가정하겠습니다.
     */

     var filename = "data.txt"
}

class DataManager {
   lazy var importer = DataImporter()
   var data = [String]()
}

let manager = DataManager()

manager.data.append("Some data")
manager.data.append("Some more data”)

// DataImporter 인스턴스는 이 시점에 생성돼 있지 않습니다.
        
print(manager.importer.filename)
// the DataImporter 인스턴스가 생성되었습니다.
// "data.txt" 파일을 출력합니다.
```

### Computed Property(연산 프로퍼티)
1. 특정 연산을 통해 필요할 때 연산을 통해 값 리턴
2. 클래스, 구조체, 열거형 모두 사용 가능
3. var로 선언
4. 반드시 연산 프로퍼티를 위한 저장 프로퍼티가 있어야 함
5. 실제 값을 가지고 있는 것이 아니라 getter, setter등을 통해서 값을 설정하고 전달
6. set만 구현은 불가능

```
struct Point{
    var x: Int
    var y: Int

    var oppositePoint: Point{
        set(point) {
            x = -point.x
            y = -point.y
       }
       get {
            return Point(x: -x, y: -y)
       }
   }
            
   var oppositePoint2: Point{
       set { // Swift에서의 set 함수에는 반드시 입력값이 있으므로 미지 지정한 newValue 키워드를 통하여 축약 가능하다.
           x = -newValue.x
           y = -newValue.y
       }
       get {
           return Point(x: -x, y: -y)
       }
   }
}
        
var point = Point(x: 10, y: 10)
print(point.oppositePoint) // Point(x: -10, y: -10)
print(point) // Point(x: 10, y: 10)
        
point.oppositePoint = Point(x: 10, y: 10)
print(point) // Point(x: -10, y: -10)
```

### Type Property(타입 프로퍼티)
1. 인스턴스 생성없이 객체내의 프로퍼티에 접근 가능
2. 프로퍼티를 타입 자체와 연결하는 것 을 지칭
3. 타입 프로퍼티는 저장 타입 프로퍼티와 연산 타입 프로퍼티가 있음
4. static 키워드를 이용해 정의
```

struct SomeStructure {
       static var storedTypeProperty = "Some value."
       static var computedTypeProperty: Int {
            return 1
       }
}

enum SomeEnumeration {
     static var storedTypeProperty = "Some value."
     static var computedTypeProperty: Int {
            return 6
     }
}

class SomeClass {
      static var storedTypeProperty = "Some value."
      static var computedTypeProperty: Int {
             return 27
      }

      class var overrideableComputedTypeProperty: Int {
            return 107
      }
}
        
class ChildSomeClass : SomeClass{
      // 슈퍼클래스의 특정 타입 프로터티를 재정의 가능
      override static var overrideableComputedTypeProperty: Int{
               return 2222
      }
}

// 별도의 인스턴스 생성없이 바로 ‘.’을 통해서 프로퍼티 접근 가능

print(SomeStructure.storedTypeProperty) // Prints "Some value."

SomeStructure.storedTypeProperty = "Another value."
print(SomeStructure.storedTypeProperty) // Prints "Another value."
print(SomeEnumeration.computedTypeProperty) // Prints "6"
print(SomeClass.computedTypeProperty) // Prints "27"
```
## Property Observer(프로퍼티 옵저버)
1. 새값이 설정될 때 해당 이벤트를 감지할 수 있는 옵저버를 제공
2. 프로퍼티 옵저버는 새 값이 이전 값과 같더라도 항상 호출
3. 지연 저장 프로퍼티에는 사용 X
4. 연산 프로퍼티 setter에서의 값의 변화를 감지할 수 있음

### 제공되는 옵저버 
> willSet : 값이 저장되기 바로 직전에 호출됨.<br>
> didSet : 새 값이 지정되고 난 직후에 호출됨.

```
class StepCounter {
       var totalSteps: Int = 0 {
            willSet(value) {
                 print("About to set totalSteps to \(totalSteps)")
            }
            didSet {
                if totalSteps > oldValue  {
                   print("Added \(totalSteps - oldValue) steps")
               }
           }
       }
}
        
let stepCounter = StepCounter()
// About to set totalSteps to 0

stepCounter.totalSteps = 200
// did가 먼저 새값이 지정되서 호출되고 will이 호출됨
// Added 200 steps
// About to set totalSteps to 200
        
stepCounter.totalSteps = 360
// Added 160 steps
// About to set totalSteps to 360
        
stepCounter.totalSteps = 896
// Added 536 steps
// About to set totalSteps to 896
```