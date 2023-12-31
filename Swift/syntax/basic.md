# 기본 문법

## 변수와 상수
* 변수(variable) : 값 수정 O 
* 상수(constant) : 값 수정 X 

```
// 변수
var str = hello
str = bye // 가능

// 상수 
let str1 = hello
str1 = bye // 불가능
```

## 여러가지 데이터형
* 문자열 : String
* 정수형 : Integer
* 실수형 : Double
* Boolean
```
let name = "김민석"
let num = 2000
let scode = 4.5
let isMan = true

// String Interpolation
// 데이터를 문자열로 표현 가능
let difficulty = "어렵다"
let maximumAge = 20

let message = "\(maximumAge)살도 iOS 개발은 \(difficulty)" 
// "20살도 iOS 개발은 어렵다"
```

## 배열과 딕셔너리
스위프트에서는 두 타입 모두 [ ]로 선언

```
// 배열
let age = [10,20,30]
let colors = ["red", "green", "blue"]
print(age[0])
// red

// 딕셔너리(Key: Value)
let language = ["한국" : "KR", "미국" : "EN", "일본" : "JA"]

print(language["미국"])
// EN

// 초기화
var emptyArr: [Int] = []
var emptyDic: [String: Any] = [:]
```

## Enum
서로 관계있는 값들을 모아서 표한하는 것

```
enum WeekDay: Int {
    case MON
    case TUE
    case WED
    case THU
    case FRI
    case SAT
    case SUN
}

var today: WeekDay = .MON
print(today)
//MON

today = WeekDay.FRI
print(today)
//PRI

today = WeekDay(rawValue: 2)!
print(today)
// WED 
// index 초과시 nil 에러!
```

```
enum Direction: String {
    case up
    case down = "hello"
}

var dir = Direction.up
dir.rawValue
// up, up

var dir2 = Direction.down
dir2.rawValue
// down, hello
```

연관값을 가지고 있는 형태로도 표현 가능
```
enum MediaType {
    case audio(String)
    case video(String)
}

var mp3 = MediaType.audio("mp3")
print(mp3)
// audio("mp3")

var h264 = MediaType.video("h264")
print(h264)
// video("h264")
```

## 조건문
### if
```
let age = 10
if age >= 10 && age < 20{
    print("10대")
} else if age >= 20 && age < 30{
    print("20대")
}
// 10대
```

### switch
```
enum Weather {
    case sun
    case cloud
    case rain
}
var weather = Weather.sun

switch weather{
case .sun:
    print("맑음")
case .cloud:
    print("흐림")
case .rain:
    print("비옴")
}
// 맑음

let name = "김민"
switch name{
case "김민석":
    print("김민석입니다.")
default:
    print("김민석이 아닙니다.")
}
// 김민석이 아닙니다.
```

### 삼항연산자
```
let age1 = 10
let age2 = 20
let ageSame = age1 == age2 ? "same" : "not same"
```
## 반복문
### for
```
let numRange = 1...10
// 1부터 10까지

let a = 10
for i in 0...a{
    print("num is \(i)")
}
//0 부터 10까지 출력

let ages = [3,20,60]
for age in ages {
    print("ages : \(age)")
}
// ages : 3 ...


let language = ["한국":"KR", "미국":"EN", "일본":"JA"]
for (key,value) in language{
    print("\(key)의 언어 코드는 \(value)")
}
// 한국의 언어 코드는 KR ...
```

### for
```
var num = 1
while num <= 20{
    print(num)
    num += 1
}
// 1부터 20까지 출력

//break
var count = 10
while count > 0{
    if count == 3 {
        break
    }
    print(count)
    count -= 1
}
// 10부터 4까지 출력
```

## 옵셔널
값이 있을 수도 없을 수도 있는 경우
```
var name: String?
let num: Int? = nil
// Int는 nil 입력 안하면 error

print(name)
// nil
// Expression implicitly coerced from 'String?' to 'Any' Warning

name = "minseok"
print(name)
// minseok

name = nil
// Warning 없애기
print(name ?? "없음")
// 없음
```

## 타입 확인하기
is, as, is?, as? 은 나중에
```
let char: Character = "A"
 
print(char is Character)       
// true
print(char is String)          
// false
```
## 타입 변환하기
```
let number = "10"

print(Int(number))
// 10
print(type(of: number))
// String
print(type(of: Int(number)))
// Int

//아래와 같은 변수 자체를 문자열에서 숫자로 변경은 안됨
var hello1 = "10"
// hello1 = Int(10) // 에러 발생

// 처음 선언시 타입 변경해주기
let hello2 = Int("10")
print(type(of: hello2))
// Int

// 내림으로 계산
let hello = 10.99
print(Int(hello))
//10
```

## 절대값 구하기
가능하면 abs 사용하기
magnitude는 타입이 바뀔 수도 있어서

```
let integer2 = Int(-15)
print(type(of: integer2))
// Int

let magnitude = integer2.magnitude
let absNum = abs(integer2)

print(type(of: magnitude))
// UInt
print(type(of: absNum))
// Int

print(magnitude is Int)
// False
print(absNum is Int)
// True
```